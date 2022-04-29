# Cancel Order

## Cancel Given Order

### Example JS Code
```js
await niftyConnectExchangeInst.cancelOrder_(
    [   // address[10] addrs,
        NiftyConnectExchange.address,                       // exchange address
        nftOwner,                                           // maker address
        "0x0000000000000000000000000000000000000000",       // taker address
        makerRelayerFeeRecipient,                           // maker relayer fee recipient
        "0x0000000000000000000000000000000000000000",       // taker relayer fee recipient
        TestERC721.address,                                 // nft contract address
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
        nftOwner,                                           // from
        "0x0000000000000000000000000000000000000000"        // to
    ],
    [   // uint[9] uints,
        exchangePrice,          // uint basePrice
        web3.utils.toBN(0),     // uint extra
        listtime,               // uint listingTime
        expireTime,             // uint expirationTime
        web3.utils.toBN(salt),  // uint salt
        ERC721TransferSelector, // uint merkleValidatorSelector
        tokenId,                // uint tokenId
        ERC721_AMOUNT,          // uint amount
        0,                      // uint totalLeaf
    ],
    1,                      // side (0 buy,1 sell)
    0,                      // Kind of sale (0 fixPrice buy/sell, 1 Auction) 
    sellReplacementPattern, // replacementPattern
    [],                     // staticExtradata
    [
        "0x0000000000000000000000000000000000000000000000000000000000000000", // merkle root hash, for trait-based order
        "0x0000000000000000000000000000000000000000000000000000000000000000"  // ipfs hash which contain the metadata of merkle proof, for trait-based order
    ],                      // merkleData
    {from: nftOwner}
);
```

### Parameter Details

|  name             |  type         | description                                        |
|-------------------|---------------|----------------------------------------------------|
| addrs             | address[10]   | addrs[0]: Exchange address<br /> addrs[1]: Order maker address<br /> addrs[2]: Order taker address<br />addrs[3]: Maker relayer fee recipient<br />addrs[4]: Taker relayer fee recipient<br />addrs[5]: NFT contract address<br />addrs[6]: StaticTarget address<br />addrs[7]: Payment token<br />addrs[8]: NFT owner address<br />addrs[9]: NFT recipient address|
| uints             | uint[9]       | uints[0]: BasePrice<br />uints[1]: EndPrice in the Dutch Auction<br />uints[2]: Order listing time<br />uints[3]: Order expire time<br />uints[4]: Salt, random number<br />uints[5]: NFT Transfer selector, refer to [nft transfer selector](nft-transfer-selector.md)<br />uints[6]: NFT asset token id<br />uints[7]: For ERC721, the amount must be 1. For ERC1155, its value should be no more than the nftOwner balance<br />uints[8]: The nft array length, only useful in trait-based order, for others leave it to 0|
| side              | uint8         | 0 buy, 1 sell |
| kind              | uint8         | Kind of sale (0 fixPrice buy/sell, 1 Auction) |
| replacementPattern| bytes         | Refer to [replacement pattern guide](replacement-pattern-guide.md)   |
| staticExtradata   | bytes         | Calldata of staticTarget contract |
| merkleData        | bytes32[2]    | merkleData[0]: merkle root hash, only used in trait-based order, leave it to zero for other scenarios<br />merkleData[1]: ipfs hash which contain the metadata of merkle proof. Only used in trait-based order, leave it to empty for other scenarios<br />Refer to [merkle proof guide](merkle-proof-guide.md)|

## Cancel All History Orders

Users can call `incrementNonce` to invalidate all history orders.
```js
await niftyConnectExchangeInst.incrementNonce({from: userAccount})
```