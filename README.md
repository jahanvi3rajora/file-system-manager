# file-system-manager
//A program that organises different files according to their categories. Based on the knowledge of JavaScript and NodeJS

let inputArr= process.argv.slice(2);
let fs= require("fs");
let path= require("path");

//console.log(inputArr);
//node main.js tree "directory Path"
//node main.js organize "directory Path"
//node main.js help
let command= inputArr[0];

let types={
      media:["mp4","mkv" ],
      archieves:['zip','7z','rar','tar','gz','ar','iso','xz'],
      documents:['docx','doc','pdf','xlsx','xls','odt','ods','odp','odg','odf','txt','ps','tex'],
      app:['exe','dmg','pkg','deb']
}

switch(command){
    case "tree":
        treeFn(inputArr[1]);
        break;
case "organize":
        organizeFn(inputArr[1]);
        break;
case "help":
        helpFn();
        break;
default:
    console.log("Please input right command");
    break;
}
function treeFn(dirPath){
    if(dirPath==undefined){
        treeHelper(process.cwd(), "");
        return;
    }else{
        let doesExist= fs.existsSync(dirPath);
        if(doesExist){
            
            treeHelper(dirPath, "");

        }else{
            console.log("Kindly enter the correct path");
            return; 
        }
    }
}

function treeHelper(dirPath, indent){
    //is file or folder
    let isFile= fs.lstatSync(dirPath).isFile();
    if(isFile==true){
        let fileName= path.basename(dirPath);
        console.log(indent + "|--" + fileName);
    }else{
        let dirName= path.basename(dirPath);
        console.log(indent + "|__" + dirName);
        let childrens= fs.readdirSync(dirPath);
        for(let i=0; i<childrens.length; i++){
            let childPath= path.join(dirPath, childrens[i]);
            treeHelper(childPath, indent + "\t");
        } 
    }
}

function organizeFn(dirPath){
    //1. input ->directory path given
    let destPath;
    if(dirPath==undefined){
        destPath= process.cwd();
        return;
    }else{
        let doesExist= fs.existsSync(dirPath);
        if(doesExist){
            //2. create-> organized_files -> directory
            destPath= path.join(dirPath, "organized_files");
            if(fs.existsSync(destPath)==false){
                fs.mkdirSync(destPath);
            }
        }else{
            console.log("Kindly enter the correct path");
            return; 
        }
    }
    organizeHelper(dirPath, destPath);
}

function organizeHelper(src,dest){
//3. identify categories of all the files present in that input directory
let childNames= fs.readdirSync(src);
//console.log(childNames);
for(let i=0; i<childNames.length; i++){
  let childAddress= path.join(src, childNames[i]);
  let isFile= fs.lstatSync(childAddress).isFile();
  if(isFile){
      let category= getCategory(childNames[i]);
      console.log(childNames[i], "belongs to -->", category);
      //4. copy/cut files to that organized directory inside of any of category folder
      sendFiles(childAddress,dest,category);
  }
}

}
function sendFiles(srcFilePath,dest, category){
    let categoryPath= path.join(dest, category);
    if(fs.existsSync(categoryPath)==false){
        fs.mkdirSync(categoryPath);
    }
    let fileName= path.basename(srcFilePath);
    let destFilePath= path.join(categoryPath, filename);
    fs.copyFileSync(srcFilePath, destFilePath);
    fs.unlinkSync(srcFilePath);
    console.log(fileName,"copied to", category);
}

function getCategory(name){

    let ext= path.extname(name);
    ext= ext.slice(1);
    for(let type in types){
        let cTypeArray= types[type];
        for( let i=0; i<cTypeArray.length; i++){
            if(ext== cTypeArray[i]){
                return type;
            }
        }
    }
    return "others";
}

function helpFn(){
    console.log(`
    List of all the commands:
      node main.js tree "directory Path"
      node main.js organize "directory Path"
      node main.js help
    `);
}
