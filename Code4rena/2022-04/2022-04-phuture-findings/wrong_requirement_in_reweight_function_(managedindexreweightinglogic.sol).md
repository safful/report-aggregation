## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Wrong requirement in reweight function (ManagedIndexReweightingLogic.sol)](https://github.com/code-423n4/2022-04-phuture-findings/issues/40) 

# Lines of code

 https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/ManagedIndexReweightingLogic.sol#L32
https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/interfaces/IIndexRegistry.sol#L19


# Vulnerability details

## Impact
The list of assets won't be changed after reweight because of reverted tx 

## Proof of Concept 
```require(_updatedAssets.length <= IIndexRegistry(registry).maxComponents())```
when [reweight](https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/ManagedIndexReweightingLogic.sol#L32) is not true, because as in the [doc](https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/interfaces/IIndexRegistry.sol#L19), 
```maxComponent``` is the maximum assets for an index, but ```_updatedAssets``` also contain the assets that you want to remove. So the comparision make no sense

## Tools Used
manual review 

## Recommended Mitigation Steps
Require ```assets.length() <= IIndexRegistry(registry).maxComponents()``` at the end of function instead 


