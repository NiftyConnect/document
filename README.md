# Document of Niftyconnect Protocol

This repository contains contracts which implements niftyconnect protocol. This protocol is a decentralized asset exchange protocol for [ERC721](https://eips.ethereum.org/EIPS/eip-721) and [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) running on EVM. Through this protocol, users can buy and sell `ERC721` or `ERC1155` assets without counterparty risk and lower exchange fee. All the market order are on the chain which facilitates users to achieve the best market liquidity. And more market parties will benefit from this protocol including order maker relayers and order taker relayers. Besides, this protocol natively support collection based order and trait based order.

## Content

- [Exchange Fee Distribution](./docs/exchange-fee-distribution.md)
- [Royalty Fee](./docs/royalty-fee.md)
- [Decentralized Order](docs/decentralized-order.md)
- [Fix Price List Guide](docs/fix-price-list-guide.md)
- [Dutch Auction Guide](docs/dutch-acution-guide.md)
- [Make Offer Guide](docs/make-offer-guide.md)
  - [Normal Offer Guide](docs/normal-offer-guide.md)
  - [Collection Based Offer Guide](docs/collection-based-offer-guide.md)
  - [Trait Based Offer Guide](docs/trait-based-offer-guide.md)
- [Cancel Order Guide](docs/cancel-order.md)

## Audit Report

Please refer to [PeckShield-Audit-Report-NiftyConnect](audit/PeckShield-Audit-Report-NiftyConnect-v1.0.pdf)