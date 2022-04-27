# Buy Order

## Normal Offer Order

The refereneced contracts locate in [TestERC20](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC20.sol), [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC721.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

```js
    const nftOwner = accounts[1];
    const buyer = accounts[2];
    const player0RelayerFeeRecipient = accounts[3];
    const player1RelayerFeeRecipient = accounts[4];

    const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
    const testERC721Inst = await TestERC721.deployed();
    const testERC20Inst = await TestERC20.deployed();

    const tokenId = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});

    // Step 4: Select Selector. For ERC721, both ERC721TransferSelector and ERC721SafeTransferSelector are valid
    const ERC721TransferSelector = web3.utils.toBN(0);
    const ERC721SafeTransferSelector = web3.utils.toBN(1);

    let buyReplacementPattern = generateBuyReplacementPatternForNormalOrder(false)

    // Step 2: calculate list time, expire time and generate random salt
    const listtime = Math.floor(Date.now() / 1000);
    const expireTime = web3.utils.toBN(listtime).add(web3.utils.toBN(3600)); // expire at one hour later
    const salt = "0x" + crypto.randomBytes(32).toString("hex");

    let exchangePrice = web3.utils.toBN(1e18);
    
    await niftyConnectExchangeInst.makeOrder_(
        [
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken
            "0x0000000000000000000000000000000000000000",       // from
            player1                                             // to
        ],
        [
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            ERC721TransferSelector,       // uint merkleValidatorSelector
            tokenId,                   // uint tokenId
            ERC721_AMOUNT,                // uint amount
            0,                            // uint totalLeaf
        ],
        0,                      // side
        0,                      // saleKind
        buyReplacementPattern,  // replacementPattern
        [],                     // staticExtradata
        [
            "0x0000000000000000000000000000000000000000000000000000000000000000",
            "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],                      // merkleData
        {from: player1}
    );

    // ---------------------------------------------------------------------------------------------------------

    await sleep(2 * 1000);
    await time.advanceBlock();

    const buyCalldata = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        "0x0000000000000000000000000000000000000000", // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        tokenId, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        "0x00", // bytes32 merkleRoot
        [],// bytes32[] memory merkleProof
    );

    const sellCalldata = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        nftOwner, // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        tokenId, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        "0x00", // bytes32 merkleRoot
        [],// bytes32[] memory merkleProof
    );

    const sellReplacementPattern = generateSellReplacementPatternForNormalOrder(false)

    const INVERSE_BASIS_POINT = await niftyConnectExchangeInst.INVERSE_BASIS_POINT();
    const exchangeFeeRate = await niftyConnectExchangeInst.exchangeFeeRate();
    const takerRelayerFeeShare = await niftyConnectExchangeInst.takerRelayerFeeShare();
    const makerRelayerFeeShare = await niftyConnectExchangeInst.makerRelayerFeeShare();
    const protocolFeeShare = await niftyConnectExchangeInst.protocolFeeShare();
    const protocolFeeRecipient = await niftyConnectExchangeInst.protocolFeeRecipient();

    const royaltyRegisterHubInst = await RoyaltyRegisterHub.deployed();
    const royaltyInfo = await royaltyRegisterHubInst.royaltyInfo(TestERC721.address, exchangePrice);
    const royaltyReceiver = royaltyInfo["0"]
    const royaltyAmount  = royaltyInfo["1"]

    const initPlayer0ERC20Balance = await testERC20Inst.balanceOf(nftOwner);
    const initPlayer0RelayerFeeRecipientERC20Balance = await testERC20Inst.balanceOf(player0RelayerFeeRecipient);
    const initPlayer1RelayerFeeRecipientERC20Balance = await testERC20Inst.balanceOf(player1RelayerFeeRecipient);
    const initProtocolFeeRecipientERC20Balance = await testERC20Inst.balanceOf(protocolFeeRecipient);
    const initRoyaltyReceiverERC20Balance = await testERC20Inst.balanceOf(royaltyReceiver);

    await niftyConnectExchangeInst.takeOrder_(
        [   // address[16] addrs,
            //buy
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken

            //sell
            NiftyConnectExchange.address,                          // exchange
            nftOwner,                                            // maker
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
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            tokenId,                   // uint tokenId
            //sell
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            tokenId,                   // uint tokenId
        ],
        [   // uint8[4] sidesKindsHowToCalls,
            0, 0,
            1, 0
        ],
        buyCalldata, // bytes calldataBuy,
        sellCalldata, // bytes calldataSell,
        buyReplacementPattern, // bytes replacementPatternBuy,
        sellReplacementPattern, // bytes replacementPatternSell,
        [],// bytes staticExtradataBuy,
        [],// bytes staticExtradataSell,
        "0x00",// bytes32 rssMetadata
        {from: nftOwner}
    );
```

## Collection Based Offer Order

```js
    const nftOwner = accounts[1];
    const player1 = accounts[2];
    const player0RelayerFeeRecipient = accounts[3];
    const player1RelayerFeeRecipient = accounts[4];

    const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
    const testERC721Inst = await TestERC721.deployed();

    const tokenIdIdx1 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx2 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});

    const emptyTokenId = web3.utils.toBN(0);

    const buyCalldata = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        "0x0000000000000000000000000000000000000000", // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        emptyTokenId, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        "0x00", // bytes32 merkleRoot
        [],// bytes32[] memory merkleProof
    );

    let buyReplacementPattern = generateBuyReplacementPatternForCollectionBasedOrder(false)

    let latestBlock = await web3.eth.getBlock("latest");
    let timestamp = latestBlock.timestamp;
    let expireTime = web3.utils.toBN(timestamp).add(web3.utils.toBN(3600)); // expire at one hour later
    let exchangePrice = web3.utils.toBN(1e18);
    let salt1 = "0x"+crypto.randomBytes(32).toString("hex")
    let salt2 = "0x"+crypto.randomBytes(32).toString("hex")
    await niftyConnectExchangeInst.makeOrder_(
        [
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken
            "0x0000000000000000000000000000000000000000",       // from
            player1                                             // to
        ],
        [
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt1),       // uint salt
            ERC721TransferSelector,       // uint merkleValidatorSelector
            emptyTokenId,                 // uint tokenId
            ERC721_AMOUNT,                // uint amount
            0,                            // uint totalLeaf
        ],
        0,                      // side
        0,                      // saleKind
        buyReplacementPattern,  // replacementPattern
        [],                     // staticExtradata
        [
            "0x0000000000000000000000000000000000000000000000000000000000000000",
            "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],                      // merkleData
        {from: player1}
    );
    await niftyConnectExchangeInst.makeOrder_(
        [
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken
            "0x0000000000000000000000000000000000000000",       // from
            player1                                             // to
        ],
        [
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt2),       // uint salt
            ERC721TransferSelector,       // uint merkleValidatorSelector
            emptyTokenId,                 // uint tokenId
            ERC721_AMOUNT,                // uint amount
            0,                            // uint totalLeaf
        ],
        0,                      // side
        0,                      // saleKind
        buyReplacementPattern,  // replacementPattern
        [],                     // staticExtradata
        [
            "0x0000000000000000000000000000000000000000000000000000000000000000",
            "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],                      // merkleData
        {from: player1}
    );

    // ---------------------------------------------------------------------------------------------------------

    await sleep(2 * 1000);
    await time.advanceBlock();

    const sellCalldata1 = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        nftOwner, // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        tokenIdIdx1, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        "0x00", // bytes32 merkleRoot
        [],// bytes32[] memory merkleProof
    );

    const sellReplacementPattern = generateSellReplacementPatternForCollectionBasedOrder(false)

    await niftyConnectExchangeInst.takeOrder_(
        [   // address[16] addrs,
            //buy
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken

            //sell
            NiftyConnectExchange.address,                          // exchange
            nftOwner,                                            // maker
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
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt1),        // uint salt
            emptyTokenId,                 // uint tokenId
            //sell
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt1),        // uint salt
            tokenIdIdx1,                  // uint tokenId
        ],
        [   // uint8[4] sidesKindsHowToCalls,
            0, 0,
            1, 0
        ],
        buyCalldata, // bytes calldataBuy,
        sellCalldata1, // bytes calldataSell,
        buyReplacementPattern, // bytes replacementPatternBuy,
        sellReplacementPattern, // bytes replacementPatternSell,
        [],// bytes staticExtradataBuy,
        [],// bytes staticExtradataSell,
        "0x00",// bytes32 rssMetadata
        {from: nftOwner}
    );
```

## Trait Based Offer Order

```js
    const nftOwner = accounts[1];
    const player1 = accounts[2];
    const player0RelayerFeeRecipient = accounts[3];
    const player1RelayerFeeRecipient = accounts[4];

    const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
    const testERC721Inst = await TestERC721.deployed();
    const testERC20Inst = await TestERC20.deployed();
    const merkleValidatorInst = await MerkleValidator.deployed();

    const tokenIdIdx1 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx2 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx3 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx4 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx5 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx6 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});
    const tokenIdIdx7 = await testERC721Inst.tokenIdIdx();
    await testERC721Inst.mint(nftOwner, {from: nftOwner});

    const proof1 = await merkleValidatorInst.calculateProof(tokenIdIdx1, tokenIdIdx2);
    const proof2 = await merkleValidatorInst.calculateProof(tokenIdIdx3, tokenIdIdx4);
    const proof3 = await merkleValidatorInst.calculateProof(proof1, proof2);

    const {merkleRoot, merkleProof} = generateMerkleProofAndRoot(tokenIdIdx1,
        [
            tokenIdIdx1,
            tokenIdIdx2,
            tokenIdIdx3,
            tokenIdIdx4,
            tokenIdIdx5,
            tokenIdIdx6,
            tokenIdIdx7,
        ]
    )
    const emptyTokenId = web3.utils.toBN(0);

    const buyCalldata = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        "0x0000000000000000000000000000000000000000", // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        emptyTokenId, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        merkleRoot, // bytes32 merkleRoot
        [
            "0x0000000000000000000000000000000000000000000000000000000000000000",
            "0x0000000000000000000000000000000000000000000000000000000000000000",
            "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],          // merkleProof
    );
    const buyReplacementPattern = generateBuyReplacementPatternForTraitBasedOrder(7, false);

    let latestBlock = await web3.eth.getBlock("latest");
    let timestamp = latestBlock.timestamp;
    let expireTime = web3.utils.toBN(timestamp).add(web3.utils.toBN(3600)); // expire at one hour later
    let exchangePrice = web3.utils.toBN(1e18);
    let salt = "0x"+crypto.randomBytes(32).toString("hex")
    const approveOrderTx = await niftyConnectExchangeInst.makeOrder_(
        [
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken
            "0x0000000000000000000000000000000000000000",       // from
            player1                                             // to
        ],
        [
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            ERC721TransferSelector,       // uint merkleValidatorSelector
            emptyTokenId,                 // uint tokenId
            ERC721_AMOUNT,                // uint amount
            7,                            // uint totalLeaf
        ],
        0,                      // side
        0,                      // saleKind
        buyReplacementPattern,  // replacementPattern
        [],                     // staticExtradata
        [
            merkleRoot,
            "0x0000000000000000000000000000000000000000000000000000000000000000"
        ],                      // merkleData
        {from: player1}
    );

    truffleAssert.eventEmitted(approveOrderTx, "OrderApprovedPartTwo",(ev) => {
        return ev.calldata.toString() === buyCalldata.toString();
    });

    // ---------------------------------------------------------------------------------------------------------

    await sleep(2 * 1000);
    await time.advanceBlock();

    const sellCalldata = await niftyConnectExchangeInst.buildCallData(
        ERC721TransferSelector, // uint selector,
        nftOwner, // address from,
        player1, // address to,
        TestERC721.address,// address nftAddress,
        tokenIdIdx1, // uint256 tokenId,
        ERC721_AMOUNT,// uint256 amount,
        merkleRoot, // bytes32 merkleRoot
        merkleProof,// bytes32[] memory merkleProof
    );

    const sellReplacementPattern = generateSellReplacementPatternForTraitBasedOrder(7, false)

    await niftyConnectExchangeInst.takeOrder_(
        [   // address[16] addrs,
            //buy
            NiftyConnectExchange.address,                          // exchange
            player1,                                            // maker
            "0x0000000000000000000000000000000000000000",       // taker
            player1RelayerFeeRecipient,                         // makerRelayerFeeRecipient
            "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
            TestERC721.address,                                 // nftAddress
            "0x0000000000000000000000000000000000000000",       // staticTarget
            TestERC20.address,                                  // paymentToken

            //sell
            NiftyConnectExchange.address,                          // exchange
            nftOwner,                                            // maker
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
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            emptyTokenId,                 // uint tokenId
            //sell
            exchangePrice,                // uint basePrice
            web3.utils.toBN(0),     // uint extra
            timestamp,                    // uint listingTime
            expireTime,                   // uint expirationTime
            web3.utils.toBN(salt),        // uint salt
            tokenIdIdx1,                  // uint tokenId
        ],
        [   // uint8[4] sidesKindsHowToCalls,
            0, 0,
            1, 0
        ],
        buyCalldata, // bytes calldataBuy,
        sellCalldata, // bytes calldataSell,
        buyReplacementPattern, // bytes replacementPatternBuy,
        sellReplacementPattern, // bytes replacementPatternSell,
        [],// bytes staticExtradataBuy,
        [],// bytes staticExtradataSell,
        "0x00",// bytes32 rssMetadata
        {from: nftOwner}
    );
```
