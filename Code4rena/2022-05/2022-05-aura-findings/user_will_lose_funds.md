## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [User will lose funds](https://github.com/code-423n4/2022-05-aura-findings/issues/108) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/contracts/AuraClaimZap.sol#L224-L226


# Vulnerability details

## Impact
It was observed that User will lose funds due to missing else condition

## Proof of Concept

1. User call claimRewards at ClaimZap.sol#L103 with Options.LockCvx as false
2. claimRewards internally calls _claimExtras
3. Everything goes good until AuraClaimZap.sol#L218

```
if (depositCvxMaxAmount > 0) {
            uint256 cvxBalance = IERC20(cvx).balanceOf(msg.sender).sub(removeCvxBalance);
            cvxBalance = AuraMath.min(cvxBalance, depositCvxMaxAmount);
            if (cvxBalance > 0) {
                //pull cvx
                IERC20(cvx).safeTransferFrom(msg.sender, address(this), cvxBalance);
                if (_checkOption(options, uint256(Options.LockCvx))) {
                    IAuraLocker(locker).lock(msg.sender, cvxBalance);
                }
            }
        }
```

4. Since user cvxBalance>0 so cvxBalance is transferred from user to the contract.
5. Now since Options.LockCvx was set to false in options so if (_checkOption(options, uint256(Options.LockCvx))) does not evaluate to true and does not execute
6. This means User cvx funds are stuck in contract

## Recommended Mitigation Steps
The condition should check if user has enabled lock for cvx, otherwise cvx should not be transferred from user

```
if (depositCvxMaxAmount > 0 && _checkOption(options, uint256(Options.LockCvx))) {
          uint256 cvxBalance = IERC20(cvx).balanceOf(msg.sender).sub(removeCvxBalance);
          cvxBalance = AuraMath.min(cvxBalance, depositCvxMaxAmount);
          if (cvxBalance > 0) {
              //pull cvx
              IERC20(cvx).safeTransferFrom(msg.sender, address(this), cvxBalance);

                  IAuraLocker(locker).lock(msg.sender, cvxBalance);
          }
      }
```

