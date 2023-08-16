## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fixed Amount of Gas Sent in Call May Be Insufficient](https://github.com/code-423n4/2022-03-joyn-findings/issues/8) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L248-L257


# Vulnerability details

## Impact

The function `attemptETHTransfer()` makes a call with a fixed amount of gas, 30,000. If the receiver is a contract this may be insufficient to process the `receive()` function. As a result the user would be unable to receive funds from this function.

## Proof of Concept

```
    function attemptETHTransfer(address to, uint256 value)
        private
        returns (bool)
    {
        // Here increase the gas limit a reasonable amount above the default, and try
        // to send ETH to the recipient.
        // NOTE: This might allow the recipient to attempt a limited reentrancy attack.
        (bool success, ) = to.call{value: value, gas: 30000}("");
        return success;
    }
```

## Recommended Mitigation Steps

Consider removing the `gas` field to use the default amount and protect from reentrancy by using reentrancy guards and the [check-effects-interaction pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html). Note this pattern is already applied correctly.

