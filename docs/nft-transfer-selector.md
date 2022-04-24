# NFT Transfer Selector

```js
const ERC721TransferSelector = web3.utils.toBN(0);
const ERC721SafeTransferSelector = web3.utils.toBN(1);
const ERC1155SafeTransferSelector = web3.utils.toBN(2);
```

```js
interface IERC721 {
    function safeTransferFrom(address from, address to, uint256 tokenId) external;
    function transferFrom(address from, address to, uint256 tokenId) external;
}
```

```js
interface IERC1155 {
    function safeTransferFrom(address from, address to, uint256 tokenId, uint256 amount, bytes data) external;
}
```