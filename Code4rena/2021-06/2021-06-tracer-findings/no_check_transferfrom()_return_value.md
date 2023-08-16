## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No check transferFrom() return value](https://github.com/code-423n4/2021-06-tracer-findings/issues/115) 

# Handle

s1m0


# Vulnerability details

## Impact
The smart contract doesn't check the return value of token.transfer() and token.transferFrom(), some erc20 token might not revert in case of error but return false.
In the [TracerPerpetualSwaps:deposit](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L151) and [Insurance:deposit](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L51) this would allow a user to deposit for free.
Other places:
[TracerPerpetualSwaps: withdraw](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L203)
[TracerPerpetualSwaps:withdrawFees](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L514)
[SafetyWithdraw:withdrawERC20Token](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/SafetyWithdraw.sol#L13)
[Insurance:withdraw](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L97)

## Recommended Mitigation Steps
Wrap the call into a require() or use openzeppelin's [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) library.

