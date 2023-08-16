## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [LP Tokens May Be Locked in Contract Due to `allowEmergencyWithdraw()` in Stage 3](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/169) 

# Handle

kirk-baird


# Vulnerability details

## Impact

The function [allowEmergencyWithdraw()](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L520) may be called by the `rocketJoeFactory.owner()` at any time. If it is called while the protocol is in Stage 3 and a pair has been created then the LP tokens will be locked and both issues and depositors will be unable to withdraw.

## Proof of Concept

If `allowEmergencyWithdraw()`  is called `stopped` is set to `true`. As a result functions [withdrawIncentives()](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L468) and [withdrawLiquidity()](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L438) will revert due to the `isStopped(false)` modifier reverting.

Additionally, [emergencyWithdraw()](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L494) will revert since all the `WAVAX` and `token` balances have been transferred to the liquidity pool.

Thus, depositors and issuers will have no methods of removing their LP tokens or incentives.


## Recommended Mitigation Steps

Consider adding the requirement `require(address(pair) != address(0), "LaunchEvent: pair not created");` to the function `allowEmergencyWithdraw()`.

