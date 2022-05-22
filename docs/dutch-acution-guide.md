# Dutch Auction Guide

## Introduction

The Dutch auction order is used to sell a nft asset with a given start price and end price. The actual match price will decrease linearly.

## Dutch Auction And Buy

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

The referenced contracts are located in [TestERC20](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC20.sol), [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC721.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

```js
const nftOwner = accounts[1];
const makerRelayerFeeRecipient = accounts[2];

const niftyConnectExchangeInst = await NiftyConnectExchange.deployed();
const testERC721Inst = await TestERC721.deployed();

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
const extraPrice = web3.utils.toBN(1e17);
const ERC721_AMOUNT = web3.utils.toBN(1); // ERC721 amount must be 1

// Step 4: Select Selector. For ERC721
const ERC721TransferSelector = web3.utils.toBN(0);
const ERC721SafeTransferSelector = web3.utils.toBN(1);

// Step 5: Calculate replacement pattern
const sellReplacementPattern = generateBuyReplacementPatternForNormalOrder(false)

// Step 6: make order
await niftyConnectExchangeInst.makeOrder_(
    [
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        "0x0000000000000000000000000000000000000000",       // taker
        makerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
        nftOwner,                                           // from
        "0x0000000000000000000000000000000000000000"        // to
    ],
    [
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        listtime,                     // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        ERC721TransferSelector,       // uint merkleValidatorSelector
        tokenId,                      // uint tokenId
        ERC721_AMOUNT,                // uint amount
        0,                            // uint totalLeaf
    ],
    1,                      // side (0 buy,1 sell)
    1,                      // Kind of sale (0 fixPrice buy/sell, 1 Auction) 
    sellReplacementPattern, // replacementPattern
    [],                     // staticExtradata
    [
        "0x0000000000000000000000000000000000000000000000000000000000000000", // merkle root hash, only for trait-based order 
        "0x0000000000000000000000000000000000000000000000000000000000000000"  // ipfs hash which contain the metadata of merkle proof, only for trait-based order 
    ],                      // merkleData
    {from: nftOwner}
);

const currentPrice = await niftyConnectExchangeInst.calculateCurrentPrice_(
    [
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        "0x0000000000000000000000000000000000000000",       // taker
        makerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
        nftOwner,                                           // from
        "0x0000000000000000000000000000000000000000"        // to
    ],
    [
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        timestamp,                    // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        ERC721TransferSelector,       // uint merkleValidatorSelector
        tokenId,                      // uint tokenId
        ERC721_AMOUNT,                // uint amount
        0,                            // uint totalLeaf
    ],
    1,                      // side
    1,                      // saleKind
    sellReplacementPattern, // replacementPattern
    [],                     // staticExtradata
    "0x00",                 // merkleRoot
);

console.log("currentPrice: "+currentPrice.toString());

// Take Order

const buyer = accounts[3];
const takerRelayerFeeRecipient = accounts[4];

// Step 1: parse order calldata from event OrderApprovedPartTwo
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
        // buy order
        NiftyConnectExchange.address,                       // exchange
        buyer,                                              // maker
        "0x0000000000000000000000000000000000000000",       // taker
        "0x0000000000000000000000000000000000000000",       // makerRelayerFeeRecipient
        takerRelayerFeeRecipient,                           // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken

        // sell order
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        "0x0000000000000000000000000000000000000000",       // taker
        makerRelayerFeeRecipient,                           // makerRelayerFeeRecipient
        "0x0000000000000000000000000000000000000000",       // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken
    ],
    [   // uint[12] uints,
        // buy order
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        listtime,                     // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenId,                      // uint tokenId
        // sell order
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        listtime,                     // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenId,                      // uint tokenId
    ],
    [   // uint8[4] sidesKindsHowToCalls,
        0, 1,
        1, 1
    ],
    buyCalldata, // bytes calldataBuy,
    sellCalldata, // bytes calldataSell,
    buyReplacementPattern, // bytes replacementPatternBuy,
    sellReplacementPattern, // bytes replacementPatternSell,
    [],// bytes staticExtradataBuy,
    [],// bytes staticExtradataSell,
    "0x00",// bytes32 rssMetadata
    {from: buyer}
);
```
