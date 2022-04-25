# Take Order


## Example
```js
        await niftyConnectExchangeInst.takeOrder_(
            [   // address[16] addrs,
                //buy order
                NiftyConnectExchange.address,                       // exchange
                player1,                                            // maker
                "0x0000000000000000000000000000000000000000",       // taker
                player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
                "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
                TestERC721.address,                                 // nftAddress
                "0x0000000000000000000000000000000000000000",       // staticTarget
                TestERC20.address,                                  // paymentToken

                //sell order
                NiftyConnectExchange.address,                       // exchange
                player0,                                            // maker
                player1,                                            // taker
                "0x0000000000000000000000000000000000000000",       // makerRelayerFeeRecipient
                player0RelayerFeeRecipient,                         // takerRelayerFeeRecipient
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
                emptyTokenId,                 // uint tokenId
                //sell
                exchangePrice,                // uint basePrice
                web3.utils.toBN(0),           // uint extra
                timestamp,                    // uint listingTime
                expireTime,                   // uint expirationTime
                web3.utils.toBN(salt),        // uint salt
                tokenIdIdx2,                  // uint tokenId
            ],
            [   // uint8[4] sidesKinds,
                0, 0,
                1, 0
            ],
            buyCalldata, // bytes calldataBuy,
            sellCalldataNew, // bytes calldataSell,
            buyReplacementPattern, // bytes replacementPatternBuy,
            sellReplacementPattern, // bytes replacementPatternSell,
            [],// bytes staticExtradataBuy,
            [],// bytes staticExtradataSell,
            "0x0000000000000000000000000000000000000000000000000000000000000000",
                // bytes32 rssMetadata, hex encoding ipfsHash
            {from: player0}
        );
```