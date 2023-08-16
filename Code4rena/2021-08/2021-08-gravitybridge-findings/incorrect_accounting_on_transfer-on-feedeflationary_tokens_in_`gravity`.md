## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect accounting on transfer-on-fee/deflationary tokens in `Gravity`](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/62) 

# Handle

shw


# Vulnerability details

## Impact

The `sendToCosmos` function of `Gravity` transfers `_amount` of `_tokenContract` from the sender using the function `transferFrom`. If the transferred token is a transfer-on-fee/deflationary token, the actually received amount could be less than `_amount`. However, since `_amount` is passed as a parameter of the `SendToCosmosEvent` event, the Cosmos side will think more tokens are locked on the Ethereum side.

## Proof of Concept

Referenced code:
[Gravity.sol#L535](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L535)
[Gravity.sol#L541](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L541)

## Recommended Mitigation Steps

Consider getting the received amount by calculating the difference of token balance (using `balanceOf`) before and after the `transferFrom`.

