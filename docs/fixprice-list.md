# Fix Price List


## Example

```js
        await niftyConnectExchangeInst.makeOrder_(
            [
                NiftyConnectExchange.address,                       // exchange
                player0,                                            // maker
                "0x0000000000000000000000000000000000000000",       // taker
                player0RelayerFeeRecipient,                         // makerRelayerFeeRecipient
                "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
                TestERC721.address,                                 // nftAddress
                "0x0000000000000000000000000000000000000000",       // staticTarget
                "0x0000000000000000000000000000000000000000",       // paymentToken
                player0,                                            // from
                "0x0000000000000000000000000000000000000000"        // to
            ],
            [
                exchangePrice,          // uint basePrice
                web3.utils.toBN(0),     // uint extra
                timestamp,              // uint listingTime
                expireTime,             // uint expirationTime
                web3.utils.toBN(salt),  // uint salt
                ERC721TransferSelector, // uint merkleValidatorSelector
                tokenIdIdx,             // uint tokenId
                ERC721_AMOUNT,          // uint amount
                0,                      // uint totalLeaf
            ],
            1,                      // side
            0,                      // saleKind
            sellReplacementPattern, // replacementPattern
            [],                     // staticExtradata
            [
                "0x0000000000000000000000000000000000000000000000000000000000000000",
                "0x0000000000000000000000000000000000000000000000000000000000000000"
            ],                      // merkleData
            {from: player0}
        );
```