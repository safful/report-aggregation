## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [AaveVault does not update TVL on deposit/withdraw](https://github.com/code-423n4/2021-12-mellow-findings/issues/41) 

# Handle

cmichel


# Vulnerability details

Aave uses **rebasing** tokens which means the token balance `aToken.balanceOf(this)` increases over time with the accrued interest.
The `AaveVault.tvl` uses a cached value that needs to be updated using a `updateTvls` call.
This call is not done when depositing tokens which allows an attacker to deposit tokens, get a fair share _of the old tvl_, update the tvl to include the interest, and then withdraw the LP tokens receiving a larger share of the _new tvl_, receiving back their initial deposit + the share of the interest.
This can be done risk-free in a single transaction.

## POC
- Imagine an Aave Vault with a single vault token, and current TVL = `1,000 aTokens`
- Attacker calls `LPIssuer.push([1000])`. This loads the old, cached `tvl`. No `updateTvl` is called.
- The `1000` underlying tokens are already balanced as there's only one aToken, then the entire amount is pushed: `aaveVault.transferAndPush([1000])`. This deposists `1000` underlying tokens to the Aave lending pool and returns `actualTokenAmounts = [1000]`. **After that** the internal `_tvls` variable is updated with the latest aTokens. This includes the 1000 aTokens just deposited **but also the new rebased aToken amounts**, the interest the vault received from supplying the tokens since last `updateTvls` call. `_tvls = _tvls + interest + 1000`
- The LP amount to mint `amountToMint` is still calculated on the old cached `tvl` memory variable, i.e., attacker receives `amount / oldTvl = 1000/1000 = 100%` of existing LP supply
- Attacker withdraws the LP tokens for 50% of the new TVL (it has been updated in `deposit`'s `transferAndPush` call). Attacker receives `50% * _newTvl = 50% * (2,000 + interest) = 1000 + 0.5 * interest`.
- Attacker makes a profit of `0.5 * interest`

## Impact
The interest since the last TVL storage update can be stolen as Aave uses rebasing tokens but the tvl is not first recomputed when depositing.
If the vaults experience low activity a significant amount of interest can accrue which can all be captured by taking a flashloan and depositing and withdrawing a large amount to capture a large share of this interest

## Recommended Mitigation Steps
Update the tvl when depositing and withdrawing before doing anything else.


