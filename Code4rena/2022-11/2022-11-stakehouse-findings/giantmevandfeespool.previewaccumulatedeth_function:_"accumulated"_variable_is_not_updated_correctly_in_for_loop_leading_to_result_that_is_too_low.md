## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-15

# [GiantMevAndFeesPool.previewAccumulatedETH function: "accumulated" variable is not updated correctly in for loop leading to result that is too low](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/160) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L82
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L91


# Vulnerability details

## Impact
The `GiantMevAndFeesPool.previewAccumulatedETH` function ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L82](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L82)) allows to view the ETH that is accumulated by an address.  

However the formula is not correct.  

In each iteration of the foor loop, `accumulated` is assigned a new value ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L91](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L91)) when actually the value should be updated like this:  
```solidity
accumulated += StakingFundsVault(payable(_stakingFundsVaults[i])).batchPreviewAccumulatedETH(
        address(this),
        _lpTokens[i]
    );
```

Obviously the `accumulated` value must be calculated for all stakingFundVaults not only for one stakingFundsVault.  

While this calculation is not used internally by the contract, it will cause any third-party contract that relies on this calculation to behave incorrectly.  

For example a third party smart contract might only allow users to withdraw once the value returned by `previewAccumulatedETH` reaches a certain threshold. Because of the issue however the accumulated ETH value that is returned will always be too low.  

## Tools Used
VSCode

## Recommended Mitigation Steps
Fix:  
```solidity
@@ -88,7 +88,7 @@ contract GiantMevAndFeesPool is ITransferHookProcessor, GiantPoolBase, Syndicate
 
         uint256 accumulated;
         for (uint256 i; i < _stakingFundsVaults.length; ++i) {
-            accumulated = StakingFundsVault(payable(_stakingFundsVaults[i])).batchPreviewAccumulatedETH(
+            accumulated += StakingFundsVault(payable(_stakingFundsVaults[i])).batchPreviewAccumulatedETH(
                 address(this),
                 _lpTokens[i]
             );
```