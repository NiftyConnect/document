# Document of Niftyconnect Protocol

This repository contains contracts which implements niftyconnect protocol. This protocol is a decentralized asset exchange protocol for ERC721 and ERC1155 running on EVM. Through this protocol, users can buy and sell ERC721 or ERC1155 assets without counterparty risk and lower exchange fee. All the market order are on the chain which facilitates users to achieve the best market liquidity. And more market parties will benefit from this protocol including order maker relayers and order taker relayers. Besides, this protocol natively support collection based order and trait based order.

## Content

- [Exchange Fee Distribution](./docs/exchange-fee-distribution.md)
- [NFT Transfer Selector](./docs/nft-transfer-selector.md)
- [Make Decentralized Order](./docs/decentralized-order.md)
    - [Sell Order: Fix Price List](./docs/fixprice-list.md)
    - [Buy Order: Make Offer](./docs/make-offer.md)
        - [Collection Based Order](./docs/collection-based-order.md)
        - [Trait Based Order](./docs/trait-based-order.md)
        - [Merkle Proof](./docs/merkle-proof.md)
    - [Dutch Auction](./docs/dutch-auction.md)
- [Take Order](./docs/take-order.md)
- [Royalty Fee](./docs/royalty-fee.md)
- [Governance](./docs/governance.md)
