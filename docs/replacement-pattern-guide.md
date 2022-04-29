# Replacement Pattern Guide


## generateBuyReplacementPatternForNormalOrder

```js
function generateBuyReplacementPatternForNormalOrder(isERC1155) {
    let buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }
    return buyReplacementPattern;
}
```


## generateSellReplacementPatternForNormalOrder
```js
function generateSellReplacementPatternForNormalOrder(isERC1155) {
    let sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "0000000000000000000000000000000000000000000000000000000000000000" +  // from
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "0000000000000000000000000000000000000000000000000000000000000000" +  // from
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }

    return sellReplacementPattern;
}
```

## generateBuyReplacementPatternForCollectionBasedOrder
```js
function generateBuyReplacementPatternForCollectionBasedOrder(isERC1155) {
    let buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }
    return buyReplacementPattern;
}
```

## generateSellReplacementPatternForCollectionBasedOrder
```js
function generateSellReplacementPatternForCollectionBasedOrder(isERC1155) {
    let sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "0000000000000000000000000000000000000000000000000000000000000000" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "0000000000000000000000000000000000000000000000000000000000000000" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }
    return sellReplacementPattern;
}
```

## generateBuyReplacementPatternForTraitBasedOrder
```js
function generateBuyReplacementPatternForTraitBasedOrder(totalLeaf, isERC1155) {
    let buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }

    if (totalLeaf>=2) {
        const merkleProofLength = calculateProofLength(totalLeaf);
        for(let idx=0; idx<merkleProofLength;idx++) {
            buyReplacementPattern = Buffer.concat([
                buyReplacementPattern,
                Buffer.from(web3.utils.hexToBytes("0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"))
            ]);
        }
    }
    return buyReplacementPattern;
}
```

## generateSellReplacementPatternForTraitBasedOrder
```js
function generateSellReplacementPatternForTraitBasedOrder(totalLeaf, isERC1155) {
    let sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "0000000000000000000000000000000000000000000000000000000000000000" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "0000000000000000000000000000000000000000000000000000000000000000" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }

    if (totalLeaf>=2) {
        const merkleProofLength = calculateProofLength(totalLeaf);
        for (let idx = 0; idx < merkleProofLength; idx++) {
            sellReplacementPattern = Buffer.concat([
                sellReplacementPattern,
                Buffer.from(web3.utils.hexToBytes("0x0000000000000000000000000000000000000000000000000000000000000000"))
            ]);
        }
    }
    return sellReplacementPattern;
}
```
