## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [InfinityExchange computes gas refunds in a way where the first order's buyer pays less than the later ones](https://github.com/code-423n4/2022-06-infinity-findings/issues/82) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L149
https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L202
https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L273


# Vulnerability details

## Impact
The way the gas refunds are computed in the InfinityExchange contract, the first orders pay less than the latter ones. This causes a loss of funds for the buyers whose orders came last in the batch.

## Proof of Concept
The issue is that the `startGasPerOrder` variable is computed within the for-loop. That causes the first iterations to be lower than later ones.

Here's an example for the following line: https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L202
To make the math easy we use the following values:
```
startGas = 1,000,000
gasPerOrder = 100,000 (so fulfilling an order costs us 100,000 gas)
ordersLength = 10
```

For the 2nd order we then get:
```
startGasPerOrder = 900,000 + ((1,000,000 + 20,000 - 900,000) / 10)
startGasPerOrder = 912,000
```
For the 9th order we get:
```
startGasPerOrder = 200,000 + ((1,000,000 + 20,000 - 200,000) / 10)
startGasPerOrder = 282,000
```

The `startGasPerOrder` variable is passed through a couple of functions without any modification until it reaches a line like this: https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L231

```sol
uint256 gasCost = (startGasPerOrder - gasleft() + wethTransferGasUnits) * tx.gasprice;
```

There, the actual gas costs for the user are computed.

In our case, that would be:

```
# 2nd order
# gasleft() is 800,00 because we said that executing the order costs ~100,000 gas. At the beginning of the order, it was 900,000 so now it's 800,000. This makes the computation a little more straightforward although it's not 100% correct.
gasCost = (912,000 - 800,000 + 50,000) * 1
gasCost = 162,000

# 9th order
gasCost = (282,000 - 100,000 + 50,000) * 1
gasCost = 232,000
```

So the 2nd order's buyer pays `162,000` while the 9th order's buyer pays `232,000`.

As I said the math was dumbed down a bit to make it easier. The actual difference might not be as big as shown here. But, there is a difference.

## Tools Used
none

## Recommended Mitigation Steps
The `startGasPerOrder` variable should be initialized *outside* the for-loop.

