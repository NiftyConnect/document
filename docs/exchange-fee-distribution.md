# Exchange Fee And Fee Distribution

## Exchange Fee

```js
uint public constant INVERSE_BASIS_POINT = 10000;
uint public exchangeFeeRate = 0;
```

The exchange fee rate is predefined in the protocol. Users can't specify this in their orders. Once an order is token by someone, the exchange fee will be deducted automatically.

```
exchangeFeeAmount = exchangePrice * exchangeFeeRate / INVERSE_BASIS_POINT
```

Initially, the `exchangeFeeRate` will be set to zero. Later, we will import governance mechanism to modify exchange fee rate.

## Fee Distribution

### Order Maker Relayer Benefit

```js
uint public constant INVERSE_BASIS_POINT = 10000;
uint public makerRelayerFeeShare = 8000;
```

Order maker relayers are the tools which facilitate users to make buy or sell orders on the protocol. These tools can specify their own addresses to gain the benefit.
```
orderMakerRelayerBenefitAmount = exchangeFeeAmount * makerRelayerFeeShare / INVERSE_BASIS_POINT
```

### Order Taker Relayer Benefit

```js
uint public constant INVERSE_BASIS_POINT = 10000;
uint public takerRelayerFeeShare = 1500;
```

Order taker relayers are the tools which facilitate users to take buy or sell orders on the protocol. These tools can specify their own addresses to gain the benefit.
```
orderTakerRelayerBenefitAmount = exchangeFeeAmount * takerRelayerFeeShare / INVERSE_BASIS_POINT
```

### Protocol Fund

```js
uint public constant INVERSE_BASIS_POINT = 10000;
uint public protocolFeeShare = 500;
address public protocolFeeRecipient;
```

The `protocolFeeRecipient` is an address owned by the protocol developer team. They can gain benefit from the exchange fee.
```
protocolBenefitAmount = exchangeFeeAmount * protocolFeeShare / INVERSE_BASIS_POINT
```

## Governance

```js
uint public exchangeFeeRate = 0;
uint public makerRelayerFeeShare = 8000;
uint public takerRelayerFeeShare = 1500;
uint public protocolFeeShare = 500;
```

The above parameters in the protocol can be modified by on-chain governance.
