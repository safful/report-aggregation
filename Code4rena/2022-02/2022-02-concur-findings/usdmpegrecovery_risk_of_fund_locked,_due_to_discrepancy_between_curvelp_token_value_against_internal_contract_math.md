## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [USDMPegRecovery Risk of fund locked, due to discrepancy between curveLP token value against internal contract math](https://github.com/code-423n4/2022-02-concur-findings/issues/70) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L90
https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L110
https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L73
https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/USDMPegRecovery.sol#L84


# Vulnerability details

## Impact
In `USDMPegRecovery` `deposit` and `withdraw` allow for direct deposits of a specific token (3crv or usdm)

The balances are directly changed and tracked in storage.

`provide` seems to be using the real balances (not the ones store) to provide liquidity.
Because of how curve works, you'll be able (first deposit) to provide exactly matching liquidity. 
But after (even just 1 or) multiple swaps, the pool will be slightly imbalanced, adding or removing liquidity at that point will drastically change the balances in the contract from the ones tracked in storage.

Eventually users won't be able to withdraw the exact amounts they deposited.

This will culminate with real balances not matching user deposits, sometimes to user advantage and other times to user disadvantage, ultimately to the protocol dismay.



## Proof of Concept
Deposit equal usdm and 3crv
LP
Do one trade on CRV
Withdraw the LP

The real balances are not matching the balances in storage

User tries to withdraw all their balances, inevitable revert

## Recommended Mitigation Steps
Either find a way to price the user contribution based on the LP tokens (use virtual_price)
Or simply have people deposit the LP token directly (avoiding the IL math which is a massive headache)

