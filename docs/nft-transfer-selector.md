# NFT Transfer Selector

```js
const ERC721TransferSelector = web3.utils.toBN(0);
const ERC721SafeTransferSelector = web3.utils.toBN(1);
const ERC1155SafeTransferSelector = web3.utils.toBN(2);
```

This protocol only supports [ERC721](https://eips.ethereum.org/EIPS/eip-721) and [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) assets. Users need to select proper selector according to their asset type and targe transfer method. 

```js
interface IERC721 {
    function transferFrom(address from, address to, uint256 tokenId) external;
    function safeTransferFrom(address from, address to, uint256 tokenId) external;
}
interface IERC1155 {
    function safeTransferFrom(address from, address to, uint256 tokenId, uint256 amount, bytes data) external;
}
```

| Asset Type |  Target Method     |   Selector                    |
|------------|--------------------|-------------------------------|
| ERC721     |  transferFrom      |   ERC721TransferSelector      |
| ERC721     |  safeTransferFrom  |   ERC721SafeTransferSelector  |
| ERC1155    |  safeTransferFrom  |   ERC1155SafeTransferSelector |