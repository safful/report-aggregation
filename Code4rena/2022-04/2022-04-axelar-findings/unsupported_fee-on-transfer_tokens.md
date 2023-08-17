## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unsupported fee-on-transfer tokens](https://github.com/code-423n4/2022-04-axelar-findings/issues/5) 

# Lines of code

https://github.com/code-423n4/2022-04-axelar/blob/main/src/AxelarGateway.sol#L284-L334


# Vulnerability details

## Impact
When tokenAddress is fee-on-transfer tokens, in the _burnTokenFrom function, the actual amount of tokens received by the contract will be less than the amount.
## Proof of Concept
https://github.com/code-423n4/2022-04-axelar/blob/main/src/AxelarGateway.sol#L284-L334
## Tools Used
None
## Recommended Mitigation Steps
Consider getting the received amount by calculating the difference of token balance (using balanceOf) before and after the transferFrom.


