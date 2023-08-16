## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Malt decimals inconsistency: StabilizerNode and DAO contracts use 18 as hard coded Malt decimals](https://github.com/code-423n4/2021-11-malt-findings/issues/175) 

# Handle

hyh


# Vulnerability details

## Impact

If Malt token be set to have lower decimals the incentives will be too big to be issued and DAO advance epoch and StabilizerNode auction start functions will fail, the system will have to be redeployed.

For example, if Malt was set to have 6 decimals like USDC, then 100*1e18 StabilizerNode defaultIncentive will be 100 trillions Malt.

## Proof of Concept

Now some parts of the system use ```malt.decimals()``` (SwingTrader, UniswapHandler), some (StabilizerNode, DAO) use 18.

DAO advanceIncentive:

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DAO.sol#L60


StabilizerNode defaultIncentive:

stabilize function
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L145

calls _startAuction in low exchangeRate case, minting defaultIncentive * 10**18 = 100 * 1e18 Malt to the sender as a caller fee.
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L344


## Recommended Mitigation Steps

If Malt decimals are meant to be set to 18, add a constant variable and use it across the system to save gas.

If the flexibility is desired ```malt.decimals()``` to be used, in a form of contract storage variable for gas optimization (```decimals()``` can be saved to storage once on initialization, and read from there afterwards).


