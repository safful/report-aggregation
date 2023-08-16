## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unsafe approve](https://github.com/code-423n4/2021-07-connext-findings/issues/13) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
Some ERC20 tokens like USDT require resetting the approval to 0 first before being able to reset it to another value. (See [Line 201](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code))
The `LIibERC20.approve` function does not do this - unlike OpenZeppelin's `safeApprove` implementation.

## Impact
Repeated USDT cross-chain transfers to the same user on receiving chain = ETH mainnet can fail due to this line not resetting the approval to zero first:

```
require(LibERC20.approve(txData.receivingAssetId, txData.callTo, toSend), "fulfill: APPROVAL_FAILED");
```

## Recommended Mitigation Steps
`LiibERC20.approve` should do two `approve` calls, one setting it to `0` first, then the real one.
Check OpenZeppelin's `safeApprove`.

