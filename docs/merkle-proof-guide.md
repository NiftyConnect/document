# Merkle Proof Guide

The merkle root and proof is only necessary in traid-based order. For other order type, just leave out this.

## Merkle Root Calculation Algorithm

Suppose a user filters out a set of tokenIds according to its traits, and the token id array is `[12,10,7,8,9,11,13]`. Then it needs to sort the token id array and build a binary tree:

```
               hash1234(hash12,hash34)
                  /            \
      hash12(hash1,hash2)   hash34(hash3,hash4)
       /          \             /             \
  hash1(7,8)  hash2(9,10)  hash3(11,12)  hash4(13,13)
   /   \       /    \        /    \           ||
  7     8      9    10      11    12          13 
```

### MerkleRoot

The merkleRoot is `hash1234(hash12,hash34)` and here is value its `0x72da2598329de27c7d20ea24372ca2a4732aaac2d9386467373baa7874b870f6`.

### MerkleProof

Suppose the target leaf is `7`, then the merkle proof is `[8, hash2(9,10), hash34(hash3,hash4)]`. Suppose the target leaf is `10`, then the merkle proof is `[9, hash1(7,8), hash34(hash3,hash4)]`.

## Generate Merkle Proof And Root

This is the example JS code to calculate merkle root and merkle proof.

```js
const keccak256 = require('keccak256');

function stringToBytes32(symbol) {
    let result = symbol;
    for (let i = 0; i < 64 - symbol.length; i++) {
        result = "0" + result;
    }
    return '0x'+result;
}

function calculateProofLength(totalLeaf) {
    let merkleProofLength = 0;
    let divResult = totalLeaf;
    let hasMod = false;
    for(;divResult!==0;) {
        let tempDivResult = Math.floor(divResult/2);
        if (tempDivResult*2<divResult) {
            hasMod = true;
        }
        divResult=tempDivResult;
        merkleProofLength++;
    }
    if (!hasMod) {
        merkleProofLength--;
    }
    return merkleProofLength;
}

function calculateProof(leafA, leafB) {
    if ( typeof leafB === 'undefined' ) {
        leafB = leafA
    }
    const leafABuffer = Buffer.from(web3.utils.hexToBytes(leafA));
    const leafBBuffer = Buffer.from(web3.utils.hexToBytes(leafB));
    let hash;
    if (leafA>leafB) {
        hash = "0x"+keccak256(Buffer.concat([leafBBuffer, leafABuffer])).toString('hex');
    } else {
        hash = "0x"+keccak256(Buffer.concat([leafABuffer, leafBBuffer])).toString('hex');
    }
    return hash;
}

function generateMerkleProofAndRoot(targetTokenId, tokenIds) {
    assert.isTrue(Array.isArray(tokenIds), "expect array");
    tokenIds.sort((a, b) => {
        return a - b
    });

    const merkleProofLength = calculateProofLength(tokenIds.length);

    let merkleProof = new Array(merkleProofLength);
    for(let idx=0; idx<merkleProofLength;idx++) {
        merkleProof[idx] = "0x0000000000000000000000000000000000000000000000000000000000000000"
    }

    let merkleProofInputs = new Array(tokenIds.length);
    for(let idx=0; idx<tokenIds.length;idx++) {
        merkleProofInputs[idx] = stringToBytes32(tokenIds[idx].toString(16))
    }
    let tempProofLength = Math.floor((tokenIds.length+1)/2)
    let tempProof = new Array(tempProofLength);
    let merkleRoot;
    let leaf = stringToBytes32(targetTokenId.toString(16));
    let proofIdx = 0;
    for(;tempProof.length>=1;) {
        for(let idx=0;idx<tempProof.length;idx++){
            let matched = false;
            if (leaf === merkleProofInputs[2*idx]) {
                merkleProof[proofIdx] = merkleProofInputs[2*idx+1];
                proofIdx++;
                matched = true;
            } else if (leaf === merkleProofInputs[2*idx+1]) {
                merkleProof[proofIdx] = merkleProofInputs[2*idx];
                proofIdx++;
                matched = true;
            }
            tempProof[idx] = calculateProof(merkleProofInputs[2*idx], merkleProofInputs[2*idx+1]);
            if (matched) {
                leaf = tempProof[idx];
            }
        }
        merkleProofInputs = tempProof;

        if (tempProof.length===1) {
            merkleRoot = tempProof[0];
            break;
        }

        tempProofLength = Math.floor((tempProofLength+1)/2)
        tempProof = new Array(tempProofLength);
    }
    return {merkleRoot, merkleProof};
}
```

## Metadata to IPFS

* Write tokenIds to a text file and upload it to IPFS.
* Convert IPFS hash to bytes32
* Fill `merkleRoot` to `merkleData[0]` and `ipfsHash` to `merkleData[1]`

This is the example JS code to convert the bese58 encoding ipfs hash to bytes32

```js
import bs58 from 'bs58'

function convertIpfsHashToBytes32(ipfsHash) {
  return "0x"+bs58.decode(ipfsHash).slice(2).toString('hex')
}
```

Suppose tokenIds is `[7,8,9,10,11,12,13]`, then the original ipfs hash is `QmZHbCohMvg1Pcf6rbh21DBhkCb7qCkwb37QK8xf6HQP39`, the bytes32 ipfs hash is `0xa2a7d9b6454df5a7f4809215f12cc347e9662a4cdcd69d83e8a0f2cf65e1ce4c`.
