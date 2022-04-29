
## Collection Based Offer Order
`generateBuyReplacementPatternForCollectionBasedOrder`
`generateSellReplacementPatternForCollectionBasedOrder`


```js

const Web3 = require('web3');
const crypto = require('crypto');
const keccak256 = require('keccak256');
const { expectRevert, time, expectEvent } = require('@openzeppelin/test-helpers');

const nftOwner = accounts[1];
const buyer = accounts[2];
const sellerRelayerFeeRecipient = accounts[3];
const buyerRelayerFeeRecipient = accounts[4];

const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
const testERC721Inst = await TestERC721.deployed();

const tokenIdIdx1 = await testERC721Inst.tokenIdIdx();
await testERC721Inst.mint(nftOwner, {from: nftOwner});
const tokenIdIdx2 = await testERC721Inst.tokenIdIdx();
await testERC721Inst.mint(nftOwner, {from: nftOwner});

const emptyTokenId = web3.utils.toBN(0);

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
        web3.utils.toBN(0),           // uint extra
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

// Step 1: parse order calldata from event OrderApprovedPartTwo
const orderApprovedPartTwoEvent = expectEvent.inLogs(makeOrdertx.logs, 'OrderApprovedPartTwo');
const buyCalldata = orderApprovedPartTwoEvent.args.calldata;

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