## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [ClearingHouse May Whitelist Duplicate AMMs](https://github.com/code-423n4/2022-02-hubble-findings/issues/50) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/ClearingHouse.sol#L339-L342
https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/ClearingHouse.sol#L269-L282


# Vulnerability details

## Impact

`ClearingHouse.sol` allows the Governance protocol to whitelist `AMM.sol` contracts. These contracts allow users to earn profits based on the price of a base asset against a quote asset.

It is possible to add the same `AMM` twice in the function `whitelistAmm()`. The impact is that unrealized profits will be counted multiple times. As a result the liquidation calculations will be incorrect, potentially allowing users to trade while insolvent or incorrectly liquidating solvent users.

Note `whitelistAmm()` may only be called by Governance.

## Proof of Concept

The function `getTotalNotionalPositionAndUnrealizedPnl()` will iterate over all `amms` summing the `unrealizedPnl`  and `notinoalPosition`, thus if an `amm` is repeated the `unrealizedPnl` and `notionalPosition` of that asset will be counted multiple times.

This is used in `_calcMarginFraction()` which calculates a users margin as a fraction of the total position. The margin fraction is used to determine if a user is liquitable or is allowed to open new positions.


## Recommended Mitigation Steps

Consider ensuring the `AMM` does not already exist in the list when adding a new `AMM`.

```
    function whitelistAmm(address _amm) external onlyGovernance {
        for (uint256 i; i < amm.length; i++) {
            require(amm[i] != IAMM(_amm), "AMM already whitelisted");
        }
        emit MarketAdded(amms.length, _amm);
        amms.push(IAMM(_amm));
    }
```

