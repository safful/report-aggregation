## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Collect modules can fail on zero amount transfers if treasury fee is set to zero](https://github.com/code-423n4/2022-02-aave-lens-findings/issues/62) 

# Lines of code

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/FeeCollectModule.sol#L176


# Vulnerability details

## Impact

Treasury fee can be zero, while collect modules do attempt to send it in such a case anyway as there is no check in place. Some ERC20 tokens do not allow zero value transfers, reverting such attempts.

This way, a combination of zero treasury fee and such a token set as a collect fee currency will revert any collect operations, rendering collect functionality unavailable

## Proof of Concept

Treasury fee can be set to zero:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/ModuleGlobals.sol#L109

Treasury fee transfer attempts are now done uncoditionally in all the collect modules.

Namely, FeeCollectModule, LimitedFeeCollectModule, TimedFeeCollectModule and LimitedTimedFeeCollectModule do not check the treasury fee to be send, `treasuryAmount`, before transferring:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/FeeCollectModule.sol#L176

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/LimitedFeeCollectModule.sol#L194

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/TimedFeeCollectModule.sol#L190

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/collect/LimitedTimedFeeCollectModule.sol#L205

The same happens in the FeeFollowModule:

https://github.com/code-423n4/2022-02-aave-lens/blob/main/contracts/core/modules/follow/FeeFollowModule.sol#L90

## References

Some ERC20 tokens revert on zero value transfers:

https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers

## Recommended Mitigation Steps

Consider checking the treasury fee amount and do transfer only when it is positive.

Now:
```
IERC20(currency).safeTransferFrom(follower, treasury, treasuryAmount);
```

To be:
```
if (treasuryAmount > 0)
	IERC20(currency).safeTransferFrom(follower, treasury, treasuryAmount);
```


