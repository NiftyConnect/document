# Replacement Pattern Guide

Order calldata is the abi encoding contract calldata to [MerkleValidator](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/MerkleValidator.sol) which specifies how the nft assets will be transferred. `MerkleValidator` has implemented three methods to handle transferring `ERC721` and `ERC1155` assets: `matchERC721UsingCriteria`, `matchERC721WithSafeTransferUsingCriteria` and `matchERC1155UsingCriteria`:

```js
    function matchERC721UsingCriteria(
        address from,
        address to,
        address token,
        uint256 tokenId,
        bytes32 root,
        bytes32[] proof
    ) external returns (bool) {}
    function matchERC721WithSafeTransferUsingCriteria(
        address from,
        address to,
        address token,
        uint256 tokenId,
        bytes32 root,
        bytes32[] proof
    ) external returns (bool) {}
    function matchERC1155UsingCriteria(
        address from,
        address to,
        address token,
        uint256 tokenId,
        uint256 amount,
        bytes32 root,
        bytes32[] proof
    ) external returns (bool) {}
```

The calldata in buy order and sell order may not be the same. However, when matching a sell order and a buy order, their calldata must be the same. Usually, sell order calldata needs to copy the nft recipient address from buy order calldata, and buy order calldata needs to copy the nft owner address from sell calldata. For trait-based order, sell order calldata need to copy the merkle proof from the buy order calldata. Here the replacement pattern is used to specify which  segment to copy.

## Replacement Pattern for Normal Order

Here normal order includes these scenarios:

* Fix price sell order.
* Fix price buy order to a given nft asset.
* Dutch auction sell order.

## Buy Order

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

## Sell Order

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

## Replacement Pattern for Collection Based Order

### Buy Order

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

## Sell Order

```js
function generateSellReplacementPatternForCollectionBasedOrder(isERC1155) {
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

## Replacement Pattern for Trait Based Order

### Buy Order

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

### Sell Order

```js
function generateSellReplacementPatternForTraitBasedOrder(totalLeaf, isERC1155) {
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
