# Merkle Proof


## Example JS code to calculate Merkle Root and Merkle Proof

```js
const keccak256 = require('keccak256');

function stringToBytes32(symbol) {
    let result = symbol;
    for (let i = 0; i < 64 - symbol.length; i++) {
        result = "0" + result;
    }
    return '0x'+result;
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
    let merkleProofLength = 0;
    let divResult = Math.floor(tokenIds.length/2)
    for(;divResult!==0;divResult=Math.floor(divResult/2)) {
        merkleProofLength++;
    }
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