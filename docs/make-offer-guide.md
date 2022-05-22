# Make Offer Guide

Users can generate makeOffer orders to buy some nft assets. If the offer price is reasonable for a target nft owner, the nft owner can accept the makeOffer order. There are three kinds of makeOffer orders: normal offer order, collection based offer, trait based offer.

## Normal Offer Order

If a user is trying to buy a given nft asset (the nft address and token are given), then it can generate a normal offer order. For how to generate normal offer orders and take these orders, please refer to [normal offer guide](normal-offer-guide.md)

## Collection Based Offer Order

If a user is trying to buy a nft asset from a given collection (only the nft address is given), then it can generate a collection based offer order. A collection based order can only be taken once, which means any nft owners from this collection can take this order and the order will be expired once taken. For how to generate collection based offer orders and take these orders, please refer to [collection based offer guide](collection-based-offer-guide.md)

## Trait Based Offer Order

If a user is trying to buy a nft asset from a given collection (the nft address is given, only target to partial token ids), then it can generate a trait based offer order. Just like collection based order, a trait based order can only be taken once, which means any nft owner from this collection can take this order which will expire once taken. Besides, users need to pick out the target token id array, then upload the token id to IPFS and generate merkle root (please refer to [merkle proof guide](merkle-proof-guide.md)). For how to generate trait based offer orders and take these orders, please refer to [trait based offer guide](trait-based-offer-guide.md)
