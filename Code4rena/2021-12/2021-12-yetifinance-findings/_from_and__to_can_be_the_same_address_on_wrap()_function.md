## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [_from and _to can be the same address on wrap() function](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/58) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In WJLP.sol, the wrap() function pulls in _amount base tokens from _from, then stakes them
to mint WAssets which it sends to _to.  It then updates _rewardOwner's reward tracking such that it now has the right to future yields from the newly minted WAssets.  But the function does not make sure that _from and _to are not the same address and failure to make this check in functions with transfer functionality has lead to severe bugs in other protocols since users rewards are updated on such transfers this can be used to manipulate the system. 

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L126

https://medium.com/@Knownsec_Blockchain_Lab/knownsec-blockchain-lab-i-kill-myself-monox-finance-security-incident-analysis-2dcb4d5ac8f 

## Tools Used
Manual code review 

## Recommended Mitigation Steps
require(address(_from) != address(_to), "_from and _to cannot be the same")

