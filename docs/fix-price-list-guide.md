# Fix Price List Guide

## Fix Price List And Buy

The refereneced contracts locate in [TestERC20](), [TestERC721](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/test/TestERC20.sol) and [NiftyConnectExchange](https://github.com/NiftyConnect/NiftyConnect-Contracts/blob/main/contracts/NiftyConnectExchange.sol)

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
const salt = "0x" + crypto.randomBytes(32).toString("hex");

// Step 3: set sell price and amount
const exchangePrice = web3.utils.toBN(1e18);
const ERC721_AMOUNT = web3.utils.toBN(1); // ERC721 amount must be 1

// Step 4: Select Selector. For ERC721, both ERC721TransferSelector and ERC721SafeTransferSelector are valid
const ERC721TransferSelector = web3.utils.toBN(0);
const ERC721SafeTransferSelector = web3.utils.toBN(1);

const sellReplacementPattern = generateSellReplacementPatternForNormalOrder(false)

await niftyConnectExchangeInst.makeOrder_(
    [   // address[10] addrs,
        NiftyConnectExchange.address,                       // exchange address
        nftOwner,                                           // maker address
        "0x0000000000000000000000000000000000000000",       // taker address
        makerRelayerFeeRecipient,                           // maker relayer fee recipient
        "0x0000000000000000000000000000000000000000",       // taker relayer fee recipient
        TestERC721.address,                                 // nft contract address
        "0x0000000000000000000000000000000000000000",       // staticTarget
        TestERC20.address,       // paymentToken
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

// sellCalldata equals to calldata in event OrderApprovedPartTwo
const sellCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    nftOwner, // address from,
    "0x0000000000000000000000000000000000000000", // address to,
    TestERC721.address,// address nftAddress,
    tokenId, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    "0x00", // bytes32 merkleRoot
    [],// bytes32[] memory merkleProof
);

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

const buyer = accounts[3];
const takerRelayerFeeRecipient = accounts[4];

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
    "0x0000000000000000000000000000000000000000000000000000000000000000",
    // bytes32 rssMetadata, hex encoding ipfsHash
    {from: buyer}
);
```
