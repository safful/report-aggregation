## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Airdrop Supply differs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/99) 

# Handle

fr0zn


# Vulnerability details

## Vulnerability Details
On the `AirdropDistribution.sol`, the `airdrop_supply` ([line 462](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L462)) value is set to be `20160000`. However, adding all the `airdropBalances` does show that the value should be `20159997` instead. 

## Impact
This does cause some operations on the contract to mislead in the results. This is more noticed on bigger airdropped accounts.

## Proof of Concept
Adding all the `airdropBalances` values do show the difference.

## Tools Used
Manual code review

## Recommended Mitigation Steps
The `airdrop_supply` should be reflecting the actual airdropped balance without misleading the total amount. Change the value to `20159997`.

