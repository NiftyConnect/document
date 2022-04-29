
`generateBuyReplacementPatternForTraitBasedOrder`
`generateSellReplacementPatternForTraitBasedOrder`

## Trait Based Offer Order

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
const testERC20Inst = await TestERC20.deployed();
const merkleValidatorInst = await MerkleValidator.deployed();

// Step 1: mint ERC721 token and setApprovalForAll

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

// Step 2: calculate merkle root and merkle proof

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

// Step 3: Write tokenId array to a text file, upload it to IPFS and decode IPFS hash to bytes32
const ipfsHash = "0xa2a7d9b6454df5a7f4809215f12cc347e9662a4cdcd69d83e8a0f2cf65e1ce4c";

// Step 4: Select Selector
const ERC721TransferSelector = web3.utils.toBN(0);

// Step 5: Calculate replacement pattern
const buyReplacementPattern = generateBuyReplacementPatternForTraitBasedOrder(7, false);

// Step 6: calculate list time, expire time and generate random salt
const listtime = Math.floor(Date.now() / 1000);
const expireTime = web3.utils.toBN(listtime).add(web3.utils.toBN(3600)); // expire at one hour later
const salt = "0x"+crypto.randomBytes(32).toString("hex");

// Step 7: set sell price, tokenId and amount
const exchangePrice = web3.utils.toBN(1e18);
const emptyTokenId = web3.utils.toBN(0);
const ERC721_AMOUNT = web3.utils.toBN(1); // ERC721 amount must be 1

// Step 8: makr trait-based order
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
        listtime,                     // uint listingTime
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
        ipfsHash
    ],                      // merkleData
    {from: buyer}
);

// Take Order

// Step 1: parse order calldata from event OrderApprovedPartTwo

const orderApprovedPartTwoEvent = expectEvent.inLogs(makeOrdertx.logs, 'OrderApprovedPartTwo');
const buyCalldata = orderApprovedPartTwoEvent.args.calldata;

// Step 2: generate sell order calldata
const sellCalldata = await niftyConnectExchangeInst.buildCallData(
    ERC721TransferSelector, // uint selector,
    nftOwner, // address from,
    buyer, // address to,
    TestERC721.address,// address nftAddress,
    tokenIdIdx1, // uint256 tokenId,
    ERC721_AMOUNT,// uint256 amount,
    merkleRoot, // bytes32 merkleRoot
    merkleProof,// bytes32[] memory merkleProof
);

// Step 3: Calculate replacementPattern for sellOrder
const sellReplacementPattern = generateSellReplacementPatternForTraitBasedOrder(7, false)

// Step 4: Take Order
await niftyConnectExchangeInst.takeOrder_(
    [   // address[16] addrs,
        //buy
        NiftyConnectExchange.address,                       // exchange
        buyer,                                            // maker
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
    {from: buyer}
);
```

// Suppose tokenIds is `[7,8,9,10,11,12,13]`, then the ipfs hash is `QmZHbCohMvg1Pcf6rbh21DBhkCb7qCkwb37QK8xf6HQP39`
// Convert IPFS hash to bytes32.
// The decode base58 encoding string to get bytes32 encoding ipfs hash: `0xa2a7d9b6454df5a7f4809215f12cc347e9662a4cdcd69d83e8a0f2cf65e1ce4c`.
const ipfsHash = "0xa2a7d9b6454df5a7f4809215f12cc347e9662a4cdcd69d83e8a0f2cf65e1ce4c";
