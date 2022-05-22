# Collection Based Offer Order

## Introduction

A trait based offer order is a buy order. It can create an offer to all nft assets in a given nft collection. It is up to the nft owner to decide whether to accept or not.

## Make Collection Based Offer Order And Take Order

### Select Selector.

Please refer to [nft transfer selector guide](nft-transfer-selector.md)

### Empty TokenId

```js
const emptyTokenId = web3.utils.toBN(0);
```

A collection based order offers to all the nft asset in the given nft collection. So the token id in maker order will be ignored. Here we just set it to zero.

### Calculate replacementPattern

Comparing with the replacementPattern in normal orders, here we need copy token id from sell order. So the replacementPattern buy should set the token id flag to `1`. Please find the implementation of `generateBuyReplacementPatternForCollectionBasedOrder` and `generateSellReplacementPatternForCollectionBasedOrder` in [replacement pattern guide](replacement-pattern-guide.md).

### Parse order parameters

Please refer to [order event](decentralized-order.md#event)

### Generate buy order calldata

Please refer to [build calldata](build-calldata.md)

## Make Order

Please refer to [make order](make-order-parameter.md)

## Take Order

Please refer to [take order](take-order-parameter.md)

## Example JS Code

The referenced contracts are located in [TestERC20](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC20.sol), [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC721.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

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
let salt = "0x"+crypto.randomBytes(32).toString("hex")

const makeOrdertx = await niftyConnectExchangeInst.makeOrder_(
    [
        NiftyConnectExchange.address,                       // exchange
        buyer,                                              // maker
        "0x0000000000000000000000000000000000000000",       // taker
        buyerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
        "0x0000000000000000000000000000000000000000",       // from
        buyer                                               // to
    ],
    [
        exchangePrice,                // uint basePrice
        web3.utils.toBN(0),           // uint extra
        timestamp,                    // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
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
    {from: buyer}
);

// Step 1: parse order calldata from event OrderApprovedPartTwo
const orderApprovedPartTwoEvent = expectEvent.inLogs(makeOrdertx.logs, 'OrderApprovedPartTwo');
const buyCalldata = orderApprovedPartTwoEvent.args.calldata;

const sellCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    nftOwner, // address from,
    buyer, // address to,
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
        NiftyConnectExchange.address,                       // exchange
        buyer,                                              // maker
        "0x0000000000000000000000000000000000000000",       // taker
        buyerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken

        //sell
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        buyer,                                              // taker
        "0x0000000000000000000000000000000000000000",       // makerRelayerFeeRecipient
        sellerRelayerFeeRecipient,                          // takerRelayerFeeRecipient
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
