## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Incorrect use of operator leads to arbitrary minting of GVT tokens](https://github.com/code-423n4/2021-06-gro-findings/issues/69) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The distributeStrategyGainLoss() function distributes any gains or losses generated from a harvest and is expected to be called only by valid protocol vault adaptors. It is an externally visible function and the access control is indirectly enforced on msg.sender by checking that vaultIndexes[msg.sender] is a valid index range 1-4. However the operator used in the require() is || instead of &&, which allows an arbitrary msg.sender, i.e. attacker, to bypass the check.

Scenario: An arbitrary non-vault address calling this function will get an index of 0 because of default mapping value in vaultIndexes[msg.sender] which will fail the > 0 check but pass the <= N_COINS + 1 check (N_COINS = 3) because 0 <= 4 which will allow control to go past this check. Furthermore, on L362, index=0 will underflow the -1 decrement (due to lack of SafeMath.sub and use of < 0.8.0 solc) and index will be set to (uint256_MAX - 1). This will allow execution to proceed to the else part of conditional meant for curve LP vault. Therefore, this will allow any random address to call this function with arbitrary values of gain/loss and distribute arbitrary gain/loss appearing to come from Curve vault.


## Proof of Concept

The attack control flow:
-> Controller.distributeStrategyGainLoss(ARBITRARY_HIGH_VALUE_OF_GAIN, 0)
-> index = 0 passes check for the index <= N_COINS + 1 part of predicate on L357 in Controller.sol
-> index = uint256_MAX after L362
-> gainUsd = ibuoy.lpToUsd(ARBITRARY_HIGH_VALUE_OF_GAIN); on L371 in Controller.sol
-> ipnl.distributeStrategyGainLoss(gainUsd, lossUsd, reward); on L376 in Controller.sol
-> (gvtAssets, pwrdAssets, performanceBonus) = handleInvestGain(lastGA, lastPA, gain, reward); on L254 in PnL.sol
-> performanceBonus = profit.mul(performanceFee).div(PERCENTAGE_DECIMAL_FACTOR); on L186 of PnL.sol
->  gvt.mint(reward, gvt.factor(gvtAssets), performanceBonus); on L256 in PnL.sol

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L355

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L356-L357

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L362

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L370-L371

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L376

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pnl/PnL.sol#L253-L258

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change || to && in require() on L357 of Controller.sol to prevent arbitrary addresses from going past this check. Or consider explicit access control for the authorized vault adaptors.

