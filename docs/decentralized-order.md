# Decentralized Order

## Order Schema

```js
    struct Order {
        address exchange;
        address maker;
        address taker;
        address makerRelayerFeeRecipient;
        address takerRelayerFeeRecipient;
        SaleKindInterface.Side side;
        SaleKindInterface.SaleKind saleKind;
        address nftAddress;
        uint tokenId;
        bytes calldata;
        bytes replacementPattern;
        address staticTarget;
        bytes staticExtradata;
        address paymentToken;
        uint basePrice;
        uint extra;
        uint listingTime;
        uint expirationTime;
        uint salt;
    }
```

All of the following fields must be present in an order. Some have special sentinel values.
### exchange
Address of the *NiftyConnectExchange* contract. This is a versioning mechanism, ensuring that all signed orders are only valid for a particular deployed version of the Wyvern Protocol on a particular Ethereum chain.
### maker
Order maker (who may be buying or selling the contract call — the maker/taker differentiation is strictly a matter of fees).
### taker
Order taker, if a specific address must take the order, otherwise the zero-address as a sentinel value to indicate that the order can be taken by anyone.
### makerRelayerFeeRecipient
Maker relayer fee will be paid to `makerRelayerFeeRecipient`. 
### takerProtocolFeeRecipient
Taker relayer fee will be paid to `takerProtocolFeeRecipient`.
### side
Side (buy or sell). Sell-side orders execute contract calls and receive tokens, buy-side orders purchase contract calls and pay tokens.
### saleKind
Kind of sale, `FixedPrice` or `DutchAuction`.
### nftAddress;
The nft asset address
### tokenId;
The token id of a NFT asset
### calldata
Calldata (bytes).
### replacementPattern
Mask specifying which parts of the calldata can be changed, or an empty array for no replacement.
### staticTarget
Target for `STATICCALL`, or zero-address as a sentinel value to indicate no `STATICCALL`.
### staticExtradata
Extra data for `STATICCALL` (bytes).
### paymentToken
Token used to pay for call, or the zero-address as a sentinel value for special-case unwrapped Ether.
### basePrice
Base price of the order, in units of the specified `paymentToken`.
### extra
Extra parameter for price calculation, specifying starting/ending price difference for Dutch auctions. Could be used to specify minimum bid for English auctions in the future should those be implemented.
### listingTime
Order listing Unix timestamp.
### expirationTime
Order expiration Unix timestamp or `0` as a sentinel value for no expiry.
### salt
Order salt to distinguish otherwise-identical orders.

## Event

```
event OrderApprovedPartOne(
                            bytes32 indexed hash, 
                            address exchange, 
                            address indexed maker, 
                            address taker, 
                            address indexed makerRelayerFeeRecipient, 
                            uint8 side, 
                            uint8 saleKind, 
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

In this protocol, only decentralized orders are supported. All order data can be parsed from above two events: `OrderApprovedPartOne` and `OrderApprovedPartTwo`. Users can write extra order message(such as personal contact information or the merkle proof of metadata for trait-based order) to IPFS and write the IPFS hash to `ipfsHash` in the event `OrderApprovedPartOne`.
