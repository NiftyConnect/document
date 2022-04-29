# Take Order Parameter

## Example JS Code
```js
await niftyConnectExchangeInst.takeOrder_(
    [   // address[16] addrs,
        //buy order
        NiftyConnectExchange.address,                       // exchange
        buyer,                                              // maker
        "0x0000000000000000000000000000000000000000",       // taker
        makerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken

        //sell order
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        buyer,                                              // taker
        "0x0000000000000000000000000000000000000000",       // makerRelayerFeeRecipient
        takerRelayerFeeRecipient,                           // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
    ],
    [   // uint[12] uints,
        //buy
        exchangePrice,                // uint basePrice
        web3.utils.toBN(0),           // uint extra
        timestamp,                    // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenId,                      // uint tokenId
        //sell
        exchangePrice,                // uint basePrice
        web3.utils.toBN(0),           // uint extra
        timestamp,                    // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenId,                      // uint tokenId
    ],
    [   // uint8[4] sidesKinds,
        0, 0,
        1, 0
    ],
    buyCalldata, // bytes calldataBuy,
    sellCalldata, // bytes calldataSell,
    buyReplacementPattern, // bytes replacementPatternBuy,
    sellReplacementPattern, // bytes replacementPatternSell,
    [],// bytes staticExtradataBuy,
    [],// bytes staticExtradataSell,
    "0x0000000000000000000000000000000000000000000000000000000000000000", // bytes32 rssMetadata, hex encoding ipfsHash
    {from: buyer}
);
```

## Paremeter Details

| name                  |  type         | description                                |
|-----------------------|---------------|--------------------------------------------|
| addrs                 | address[16]   | // buy order<br />addrs[0]: exchange<br /> addrs[1]: maker<br /> addrs[2]: taker<br /> addrs[3]: makerRelayerFeeRecipient<br /> addrs[4]: takerRelayerFeeRecipient<br /> addrs[5]: nftAddress<br /> addrs[6]: staticTarget<br /> addrs[7]: paymentToken<br /><br />// sell order<br />addrs[8]: exchange<br /> addrs[9]: maker<br /> addrs[10]: taker<br /> addrs[11]: makerRelayerFeeRecipient<br /> addrs[12]: takerRelayerFeeRecipient<br /> addrs[13]: nftAddress<br /> addrs[14]: staticTarget<br /> addrs[15]: paymentToken |
| uints                 | uint[12]      | // buy order<br />uints[0]: basePrice<br />uints[1]: extra, endPrice in Dutch auction<br />uints[2]: listing iime<br />uints[3]: expirationTime<br />uints[4]: salt<br />uints[5]: tokenId<br /><br />//sell order<br />uints[6]: basePrice<br />uints[7]: extra, endPrice in Dutch auction<br />uints[8]: listing iime<br />uints[9]: expirationTime<br />uints[10]: salt<br />uints[11]: tokenId |
| sidesKinds            | uint8[4]      | // buy order<br />sidesKinds[0]: side <br />sidesKinds[1]: saleKind<br /><br />// sell order<br />sidesKinds[2]: side<br />sidesKinds[3]saleKind |
| buyCalldata           | bytes         | buy order calldata  |
| sellCalldata          | bytes         | sell order calldata  |
| buyReplacementPattern | bytes         | buy order replacementPattern  |
| sellReplacementPattern| bytes         | sell order replacementPattern  |
| staticExtradataBuy    | bytes         | calldata of static target contract in buy order |
| staticExtradataSell   | bytes         | calldata of static target contract in sell order  |
