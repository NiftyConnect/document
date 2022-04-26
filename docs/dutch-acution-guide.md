# Dutch Auction Guide

## Auction Order And Buy

```js

function generateBuyReplacementPatternForNormalOrder(isERC1155) {
    let buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
        "0000000000000000000000000000000000000000000000000000000000000000" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        buyReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // from
            "0000000000000000000000000000000000000000000000000000000000000000" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }
    return buyReplacementPattern;
}

function generateSellReplacementPatternForNormalOrder(isERC1155) {
    let sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
        "0x" +
        "00000000" +                                                          // selector
        "0000000000000000000000000000000000000000000000000000000000000000" +  // from
        "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // to
        "0000000000000000000000000000000000000000000000000000000000000000" +  // token
        "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
        "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
        "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
    ));

    if (isERC1155 === true) {
        sellReplacementPattern = Buffer.from(web3.utils.hexToBytes(
            "0x" +
            "00000000" +                                                          // selector
            "0000000000000000000000000000000000000000000000000000000000000000" +  // from
            "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" +  // to
            "0000000000000000000000000000000000000000000000000000000000000000" +  // token
            "0000000000000000000000000000000000000000000000000000000000000000" +  // tokenId
            "0000000000000000000000000000000000000000000000000000000000000000" +  // amount
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleRoot
            "0000000000000000000000000000000000000000000000000000000000000000" +  // merkleProof Offset
            "0000000000000000000000000000000000000000000000000000000000000000"    // merkleProof Length
        ));
    }

    return sellReplacementPattern;
}


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

// Step 4: Select Selector. For ERC721, both ERC721TransferSelector and ERC721SafeTransferSelector are valid
const ERC721TransferSelector = web3.utils.toBN(0);
const ERC721SafeTransferSelector = web3.utils.toBN(1);

const sellReplacementPattern = generateBuyReplacementPatternForNormalOrder(false)

let latestBlock = await web3.eth.getBlock("latest");
const listtime = Math.floor(Date.now() / 1000);
const expireTime = web3.utils.toBN(listtime).add(web3.utils.toBN(10)); // expire at one minute later

let salt = "0x"+crypto.randomBytes(32).toString("hex");

await niftyConnectExchangeInst.makeOrder_(
    [
        NiftyConnectExchange.address,                       // exchange
        nftOwner,                                           // maker
        "0x0000000000000000000000000000000000000000",       // taker
        makerRelayerFeeRecipient,                         // makerRelayerFeeRecipient
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
        tokenIdIdx,                   // uint tokenId
        ERC721_AMOUNT,                // uint amount
        0,                            // uint totalLeaf
    ],
    1,                      // side
    1,                      // saleKind
    sellReplacementPattern, // replacementPattern
    [],                     // staticExtradata
    [
        "0x0000000000000000000000000000000000000000000000000000000000000000",
        "0x0000000000000000000000000000000000000000000000000000000000000000"
    ],                      // merkleData
    {from: nftOwner}
);

const sellCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    player0, // address from,
    "0x0000000000000000000000000000000000000000", // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x00", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);

const buyer = accounts[3];
const takerRelayerFeeRecipient = accounts[4];

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

const buyReplacementPattern = generateBuyReplacementPatternForNormalOrder(false)

await niftyConnectExchangeInst.takeOrder_(
    [   // address[16] addrs,
        //buy
        NiftyConnectExchange.address,                       // exchange
        buyer,                                              // maker
        "0x0000000000000000000000000000000000000000",       // taker
        "0x0000000000000000000000000000000000000000",       // makerRelayerFeeRecipient
        takerRelayerFeeRecipient,                           // takerRelayerFeeRecipient
        TestERC721.address,                                 // nftAddress
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,                                  // paymentToken

        //sell
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
        //buy
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        listtime,                     // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenIdIdx,                   // uint tokenId
        //sell
        exchangePrice,                // uint basePrice
        extraPrice,                   // uint extra
        listtime,                     // uint listingTime
        expireTime,                   // uint expirationTime
        web3.utils.toBN(salt),        // uint salt
        tokenIdIdx,                   // uint tokenId
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