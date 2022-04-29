# Fix Price List Guide

## Introduction
The fix price list order is used to sell a nft asset at a given price. Anyone who want to buy the nft asset at the given price can take the order.

## Make Fix Price List And Take Order

### Select Selector.
Please refer to [nft transfer selector guide](nft-transfer-selector.md)

### Calculate replacementPattern
Please refer to [replacement pattern guide](replacement-pattern-guide.md)

### Parse order parameters
Please refer to [order event](decentralized-order.md#event)

### Generate buy order calldata
Please refer to [build calldata](build-calldata.md)

## Make Order
Please refer to [make order](make-order-parameter.md)

## Take Order
Please refer to [take order](take-order-parameter.md)

## Example JS Code

The refereneced contracts locate in [TestERC20](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC20.sol), [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC721.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

```js

const Web3 = require('web3');
const crypto = require('crypto');
const keccak256 = require('keccak256');
const { expectRevert, time, expectEvent } = require('@openzeppelin/test-helpers');

const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
const testERC721Inst = await TestERC721.deployed();

const nftOwner = accounts[1];
const makerRelayerFeeRecipient = accounts[2];

/* Make Order */

// Step 1: Mint ERC721 token and setApprovalForAll
const tokenId = await testERC721Inst.tokenIdIdx();
await testERC721Inst.mint(nftOwner, {from: nftOwner});
await testERC721Inst.setApprovalForAll(NiftyConnectExchange.address, true, {from: nftOwner});

// Step 2: Calculate list time, expire time and generate random salt
const listtime = Math.floor(Date.now() / 1000);
const expireTime = web3.utils.toBN(listtime).add(web3.utils.toBN(3600)); // expire at one hour later
const salt = "0x" + crypto.randomBytes(32).toString("hex");

// Step 3: Set sell price and amount
const exchangePrice = web3.utils.toBN(1e18);
const ERC721_AMOUNT = web3.utils.toBN(1); // ERC721 amount must be 1

// Step 4: Select nft transfer selector
const ERC721TransferSelector = web3.utils.toBN(0);

// Step 5: Calculate replacementPattern
const sellReplacementPattern = generateSellReplacementPatternForNormalOrder(false)

// Step 6: Make Fix Price Order
await niftyConnectExchangeInst.makeOrder_(
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

/* Take Order */

const buyer = accounts[3];
const takerRelayerFeeRecipient = accounts[4];

// Step 1: parse order parameters
const orderApprovedPartTwoEvent = expectEvent.inLogs(makeOrdertx.logs, 'OrderApprovedPartTwo');
const sellCalldata = orderApprovedPartTwoEvent.args.calldata;

// Step 2: generate buy order calldata
const buyCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    "0x0000000000000000000000000000000000000000", // address from,
    buyer, // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x00", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);

// Step 3: Calculate replacementPattern for buyOrder
const buyReplacementPattern = generateBuyReplacementPatternForNormalOrder(false)

// Step 4: Take Order
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
