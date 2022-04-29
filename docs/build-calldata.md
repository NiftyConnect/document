# Build Calldata

The NiftyConnect exchange contract provide an util method to build calldata which is necessary in `makeOrder` and `takeOrder`.

```js
const sellCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    nftOwner, // address from,
    "0x0000000000000000000000000000000000000000", // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x0000000000000000000000000000000000000000000000000000000000000000", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);

const buyCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    "0x0000000000000000000000000000000000000000", // address from,
    buyer, // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x0000000000000000000000000000000000000000000000000000000000000000", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);
```

The above is js example to call the util method. 

|  name       |  type     | description                                |
|-------------|-----------|--------------------------------------------|
| selector    | uint256   |Refer to [NFT Transfer Selector](nft-transfer-selector.md)|
| from        | address   |NFTOwner address. <br />When making buy orders, leave it to zero.  |
| to          | address   |NFT recipient address. <br />When making sell orders, leave it to zero.  |
| nftAddress  | address   |NFT contract address  |
| tokenId     | uint256   |NFT asset token id |
| amount      | uint256   |For ERC721, the amount must be 1. <br />For ERC1155, its value should be no more than the nftOwner balance|
| merkleRoot  | bytes32   |Only used in trait-based order, leave it to zero for other scenarios |
| merkleProof | bytes32[] |Only used in trait-based order, leave it to empty for other scenarios |
