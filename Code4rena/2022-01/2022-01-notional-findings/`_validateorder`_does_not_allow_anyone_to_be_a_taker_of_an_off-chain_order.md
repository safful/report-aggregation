## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`_validateOrder` Does Not Allow Anyone To Be A Taker Of An Off-Chain Order](https://github.com/code-423n4/2022-01-notional-findings/issues/152) 

# Handle

leastwood


# Vulnerability details

## Impact

The `EIP1271Wallet` contract intends to allow the treasury manager account to sign off-chain orders in 0x on behalf of the `TreasuryManager` contract, which holds harvested assets/`COMP` from Notional. While the `EIP1271Wallet._validateOrder` function mostly prevents the treasury manager from exploiting these orders, it does not ensure that the `takerAddress` and `senderAddress` are set to the zero address. As a result, it is possible for the manager to have sole rights to an off-chain order and due to the flexibility in `makerPrice`, the manager is able to extract value from the treasury by maximising the allowed slippage.

By setting `takerAddress` to the zero address, any user can be the taker of an off-chain order. By setting `senderAddress` to the zero address, anyone is allowed to access the exchange methods that interact with the order, including filling the order itself. Hence, these two order addresses can be manipulated by the manager to effectively restrict order trades to themselves.

## Proof of Concept

https://github.com/0xProject/0x-monorepo/blob/0571244e9e84b9ad778bccb99b837dd6f9baaf6e/contracts/exchange-libs/contracts/src/LibOrder.sol#L66
```
address takerAddress;           // Address that is allowed to fill the order. If set to 0, any address is allowed to fill the order.
```

https://github.com/0xProject/0x-monorepo/blob/0571244e9e84b9ad778bccb99b837dd6f9baaf6e/contracts/exchange/contracts/src/MixinExchangeCore.sol#L196-L250

https://github.com/0xProject/0x-monorepo/blob/0571244e9e84b9ad778bccb99b837dd6f9baaf6e/contracts/exchange/contracts/src/MixinExchangeCore.sol#L354-L374


https://github.com/code-423n4/2022-01-notional/blob/main/contracts/utils/EIP1271Wallet.sol#L147-L188
```
function _validateOrder(bytes memory order) private view {
    (
        address makerToken,
        address takerToken,
        address feeRecipient,
        uint256 makerAmount,
        uint256 takerAmount
    ) = _extractOrderInfo(order);

    // No fee recipient allowed
    require(feeRecipient == address(0), "no fee recipient allowed");

    // MakerToken should never be WETH
    require(makerToken != address(WETH), "maker token must not be WETH");

    // TakerToken (proceeds) should always be WETH
    require(takerToken == address(WETH), "taker token must be WETH");

    address priceOracle = priceOracles[makerToken];

    // Price oracle not defined
    require(priceOracle != address(0), "price oracle not defined");

    uint256 slippageLimit = slippageLimits[makerToken];

    // Slippage limit not defined
    require(slippageLimit != 0, "slippage limit not defined");

    uint256 oraclePrice = _toUint(
        AggregatorV2V3Interface(priceOracle).latestAnswer()
    );

    uint256 priceFloor = (oraclePrice * slippageLimit) /
        SLIPPAGE_LIMIT_PRECISION;

    uint256 makerDecimals = 10**ERC20(makerToken).decimals();

    // makerPrice = takerAmount / makerAmount
    uint256 makerPrice = (takerAmount * makerDecimals) / makerAmount;

    require(makerPrice >= priceFloor, "slippage is too high");
}
```

## Tools Used

Manual code review.
Discussions with Notional team.

## Recommended Mitigation Steps

Consider adding `require(takerAddress == address(0), "manager cannot set taker");` and `require(senderAddress == address(0), "manager cannot set sender");` statements to `_validateOrder`. This should allow any user to fill an order and prevent the manager from restricting exchange methods to themselves.

