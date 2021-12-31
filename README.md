# ChainedBlock.js
```
//chainedBlock.js
const fs = require("fs");
const merkle = require("merkle"); // npm i merkle
const cryptojs = require("crypto-js"); // npm i crypto-js

class Block { //블록 헤더와 바디 생성
   constructor(header, body) {
      this.header = header;
      this.body = body;
   }
}

class BlockHeader { 
   constructor( //블록 헤더에 들어가는 값
      version, //해당 버전
      index, //인덱스
      previousBlockHash, //이전 해시값
      merkleRoot, //현재 해시값
      timestamp, //시간
      bit, //난이도
      nonce //nonce값
   ) {
      this.version = version;
      this.index = index;
      this.previousBlockHash = previousBlockHash;
      this.merkleRoot = merkleRoot;
      this.timestamp = timestamp;
      this.bit = bit;
      this.nonce = nonce;
   }
}

function getVersion() {
   const package = fs.readFileSync("package.json");
   // console.log(JSON.parse(package).version);
   return JSON.parse(package).version;
}

function createGenesisBlock() { //최초의 블록 생성
   const version = getVersion();
   const index = 0;
   const previousBlockHash = "0".repeat(64);
   const timestamp = parseInt(Date.now() / 1000); // 1000=1s

   const body = ["hello block"];
   const tree = merkle("sha256").sync(body);
   const merkleRoot = tree.root() || "0".repeat(64);
   const bit = 0;
   const nonce = 0;

   const header = new BlockHeader(
      version,
      index,
      previousBlockHash,
      merkleRoot,
      timestamp,
      bit,
      nonce
   );

   return new Block(header, body);
}

let Blocks = [createGenesisBlock()];

function getBlocks() {
   return Blocks;
}

function getLastBlock() {
   return Blocks[Blocks.length - 1];
}

function createHash(data) {
   const {
      version,
      index,
      previousBlockHash,
      merkleRoot,
      timestamp,
      bit,
      nonce,
   } = data.header;
   const blockString =
      version + index + previousBlockHash + merkleRoot + timestamp + bit + nonce;
   const hash = cryptojs.SHA256(blockString).toString(); //위에 blockString 에 담은값을 crypto모듈
   //에서SHA256을 통해 해쉬값으로 바꾼후 이걸 String으로 
   return hash; // 반환한다음
}

function nextBlock(bodyData) {
   const prevBlock = getLastBlock();//getLastBlock : Blocks[Blocks.length - 1] 블록의 마지막 값
   const version = getVersion(); //버전 불러옴
   const index = prevBlock.header.index + 1; //이전 헤더의 인덱스 +1
   const previousBlockHash = createHash(prevBlock); // 이전 블록의 해쉬값
   const timestamp = parseInt(Date.now() / 1000); //시간 1초단위
   const tree = merkle("sha256").sync(bodyData); //bodyData받아온거 해쉬값으로 변환
   const merkleRoot = tree.root() || "0".repeat(); //0을 돌려서
   const bit = 0; //난이도
   const nonce = 0; //반복한숫자

   const header = new BlockHeader(version, index, previousBlockHash, merkleRoot, timestamp, bit, nonce);
   return new Block(header, bodyData) //헤더에 정보 저장 , 헤더랑 바디 블록으로 리턴
}

function addBlock(bodyData) {
   const newBlock = nextBlock(bodyData);
   Blocks.push(newBlock); //새로운 블록을 추가
}

const genesisBlock = createGenesisBlock();
console.log(genesisBlock); //최초의블록

const block1 = nextBlock(["transaction1"])
// console.log(block1);

addBlock(['transaction2'])
addBlock(['transaction3'])
addBlock(['transaction4'])
addBlock(['transaction5'])
console.log(Blocks);
module.exports = {
	Blocks, getLastBlock, createHash, nextBlock, getVersion, getBlocks
}
```
