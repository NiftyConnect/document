# Merkle Proof



## Generate Merkle Proof And Root

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

## Generate Buy Replacement Pattern

```js
function generateBuyReplacementPattern(totalLeaf) {
    const merkleProofLength = calculateProofLength(totalLeaf);

    let buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +
        "0000000000000000000000000000000000000000000000000000000000000000" +
        "0000000000000000000000000000000000000000000000000000000000000000"
    ));

    for(let idx=0; idx<merkleProofLength;idx++) {
        buyReplacementPattern = Buffer.concat([
            buyReplacementPattern,
            Buffer.from(web3.utils.hexToBytes("0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"))
        ]);
    }
    return buyReplacementPattern;

}
```

## Generate Sell Replacement Pattern

```js

function generateSellReplacementPattern(totalLeaf) {
    const merkleProofLength = calculateProofLength(totalLeaf);

    let sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +
        "0000000000000000000000000000000000000000000000000000000000000000" + // selector
        "0000000000000000000000000000000000000000000000000000000000000000" + // from
        "0000000000000000000000000000000000000000000000000000000000000000" + // to
        "0000000000000000000000000000000000000000000000000000000000000000" + // token
        "0000000000000000000000000000000000000000000000000000000000000000" + // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +
        "0000000000000000000000000000000000000000000000000000000000000000"
    ));

    for(let idx=0; idx<merkleProofLength;idx++) {
        sellReplacementPattern = Buffer.concat([
            sellReplacementPattern,
            Buffer.from(web3.utils.hexToBytes("0x0000000000000000000000000000000000000000000000000000000000000000"))
        ]);
    }
    return sellReplacementPattern;
}
```
