# Decentralized Order

## Decentralized Order

```js
/* An order on the exchange. */
    struct Order {
        /* Exchange address, intended as a versioning mechanism. */
        address exchange;
        /* Order maker address. */
        address maker;
        /* Order taker address, if specified. */
        address taker;
        /*  Order fee recipient or zero address for taker order. */
        address makerRelayerFeeRecipient;
        /*  Taker order fee recipient */
        address takerRelayerFeeRecipient;
        /* Side (buy/sell). */
        SaleKindInterface.Side side;
        /* Kind of sale. */
        SaleKindInterface.SaleKind saleKind;
        /* nftAddress. */
        address nftAddress;
        /* nft tokenId. */
        uint tokenId;
        /* Calldata. */
        bytes calldata;
        /* Calldata replacement pattern, or an empty byte array for no replacement. */
        bytes replacementPattern;
        /* Static call target, zero-address for no static call. */
        address staticTarget;
        /* Static call extra data. */
        bytes staticExtradata;
        /* Token used to pay for the order, or the zero-address as a sentinel value for Ether. */
        address paymentToken;
        /* Base price of the order (in paymentTokens). */
        uint basePrice;
        /* Auction extra parameter - minimum bid increment for English auctions, starting/ending price difference. */
        uint extra;
        /* Listing timestamp. */
        uint listingTime;
        /* Expiration timestamp - 0 for no expiry. */
        uint expirationTime;
        /* Order salt, used to prevent duplicate hashes. */
        uint salt;
        /* NOTE: uint nonce is an additional component of the order but is read from storage */
    }
```

## Event

```
event OrderApprovedPartOne(
                            bytes32 indexed hash, 
                            address exchange, 
                            address indexed maker, 
                            address taker, 
                            address indexed makerRelayerFeeRecipient, 
                            SaleKindInterface.Side side, 
                            SaleKindInterface.SaleKind saleKind, 
                            address nftAddress, 
                            uint256 tokenId, 
                            bytes32 ipfsHash);
                            
event OrderApprovedPartTwo(
                            bytes32 indexed hash, 
                            bytes calldata, 
                            bytes replacementPattern, 
                            address staticTarget, 
                            bytes staticExtradata, 
                            address paymentToken, 
                            uint basePrice, 
                            uint extra, 
                            uint listingTime, 
                            uint expirationTime, 
                            uint salt);

```

## Extra Order Message on IPFS
