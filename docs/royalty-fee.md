# Royalty Fee

When an order is matched and the royalty fee of the referenced NFT asset is not zero, the royalty fee will be directly transferred to the royalty fee address. This protocol supports two ways to specify the royalty fee amount and recipient address.

## IERC2981

```js
interface IERC2981 {
    /// @notice Called with the sale price to determine how much royalty
    //          is owed and to whom.
    /// @param _tokenId - the NFT asset queried for royalty information
    /// @param _salePrice - the sale price of the NFT asset specified by _tokenId
    /// @return receiver - address of who should be sent the royalty payment
    /// @return royaltyAmount - the royalty payment amount for _salePrice
    function royaltyInfo(uint256 _tokenId, uint256 _salePrice) external view returns (address receiver, uint256 royaltyAmount);
}
```

Firstly, this protocol will check whether the target nft contract has implemented the above interface. If so, this protocol will follow the royalty fee mechanism specified by the nft contract. Otherwise, the protocol will try to get the royaltyInfo from `RoyaltyRegisterHub`.

## RoyaltyRegisterHub

```js
interface IOwnable {
    function owner() external view returns (address);
}
```

If a nft contract doesn't implement the interface `IERC2981`, the owners address of the NFT contract can specify its royalty parameters in `RoyaltyRegisterHub`.

```js
function setRoyaltyRateFromNFTOwners(address _nftAddress, uint256 _royaltyRate, address _receiver) public returns (bool)
```

If the `IOwnable` interface is not implemented either, the nft contract deployer can contract the protocol developer team to set the royalty parameters.
