## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [treasury is vulnerable to sandwich attack](https://github.com/code-423n4/2021-10-mochi-findings/issues/60) 

# Handle

jonah1005


# Vulnerability details

# treasury is vulnerable to sandwich attack.


## Impact
There's a permissionless function `veCRVlock` in MochiTreasury. Since everyone can trigger this function, the attacker can launch a sandwich attack with flashloan to steal the funds.
[MochiTreasuryV0.sol#L73-L94](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/treasury/MochiTreasuryV0.sol#L73-L94)

Attackers can possibly steal all the funds in the treasury. I consider this is a high-risk issue.

## Proof of Concept
[MochiTreasuryV0.sol#L73-L94](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/treasury/MochiTreasuryV0.sol#L73-L94)

Here's an exploit pattern
1. Flashloan and buy CRV the uniswap pool
2. Trigger `veCRVlock()`
3. The treasury buys CRV at a very high price.
4. Sell CRV and pay back the loan.

## Tools Used

None

## Recommended Mitigation Steps
Recommend to add `onlyOwner` modifier. 

