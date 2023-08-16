## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No sanity checks for user supplied promotion values](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/29) 

# Handle

kenzo


# Vulnerability details

User supplied values are not checked and can lead to unexpected behavior (such as division by 0, underflows...)

## Impact
I believe the high risk impact has been detailed and mitigated in other findings.
However, for cleanliness and preventive measures, I suggest not allowing illogical inputs.

## Proof of Concept
There is no validation on the user supplied promotion inputs. [(Code ref)](https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L88:#L116)
Therefore for example, a user can supply _numberOfEpochs = 0, _epochDuration = 0, _tokensPerEpoch = 0.
This leads to garbage values in the contract. A user can create a promotion without paying any tokens (if _numberOfEpochs or _tokensPerEpoch  = 0). These may confuse front ends, or compound to lead to more serious errors.

## Recommended Mitigation Steps
Add sanity checks (such as inputs > 0) to `createPromotion` and `extendPromotion`.

