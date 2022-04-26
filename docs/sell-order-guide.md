# Sell Order

## Fix Price List Order

The refereneced contracts locate in [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC721.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

```js
    const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
    const testERC721Inst = await TestERC721.deployed();

    const nftOwner = accounts[1];
    const makerRelayerFeeRecipient = accounts[2];

    // Step 1: mint ERC721 token and setApprovalForAll
    const tokenId = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    await testERC721Inst.setApprovalForAll(NiftyConnectExchange.address, true, {from: nftOwner});

    // Step 2: calculate list time, expire time and generate random salt
    const listtime = Math.floor(Date.now() / 1000);
    const expireTime = web3.utils.toBN(listtime).add(web3.utils.toBN(3600)); // expire at one hour later
    const salt = "0x"+crypto.randomBytes(32).toString("hex");
    
    // Step 3: set sell price and amount
    const exchangePrice = web3.utils.toBN(1e18);
    const ERC721_AMOUNT = web3.utils.toBN(1); // ERC721 amount must be 1

    // Step 4: Select Selector. For ERC721, both ERC721TransferSelector and ERC721SafeTransferSelector are valid
    const ERC721TransferSelector = web3.utils.toBN(0);
    const ERC721SafeTransferSelector = web3.utils.toBN(1);
    
    await niftyConnectExchangeInst.makeOrder_(
        [   // address[10] addrs,
            NiftyConnectExchange.address,                       // exchange address
            nftOwner,                                           // maker address
            "0x0000000000000000000000000000000000000000",       // taker address
            makerRelayerFeeRecipient,                           // maker relayer fee recipient
            "0x0000000000000000000000000000000000000000",       // taker relayer fee recipient
            TestERC721.address,                                 // nft contract address
            "0x0000000000000000000000000000000000000000",       // staticTarget
            "0x0000000000000000000000000000000000000000",       // paymentToken
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
            "0x0000000000000000000000000000000000000000000000000000000000000000", // merkle root hash
            "0x0000000000000000000000000000000000000000000000000000000000000000"  // ipfs hash which contain the metadata of merkle proof 
        ],                      // merkleData
        {from: nftOwner}
    );
```

- **Merkle Root Hash** must be zero bytes32("0x0000000000000000000000000000000000000000000000000000000000000000").
- **IPFS Hash** can be any value.

## Auction Order