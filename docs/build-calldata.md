# Build Calldata

```js
const buyCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    "0x0000000000000000000000000000000000000000", // address from,
    buyer, // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x00", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);
```

|  name       |  type     | description                                |
|-------------|-----------|--------------------------------------------|
| selector    | uint256   |                                            |
| from        | address   |                                            |
| to          | address   |                                            |
| nftAddress  | address   |                                            |
| tokenId     | uint256   |                                            |
| merkleRoot  | bytes32   |                                            |
| merkleProof | bytes32[] |                                            |
