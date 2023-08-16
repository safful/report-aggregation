## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Typo in PoolTemplate unlock function results in user being able to unlock multiple times](https://github.com/code-423n4/2022-01-insure-findings/issues/192) 

# Handle

loop


# Vulnerability details

The function `unlock()` in PoolTemplate has a typo where it compares `insurances[_id].status` to `false` rather than setting it to `false`. If the conditions are met to unlock the funds for an id, the user should be able to call the `unlock()` function once for that id as `insurances[_id].amount` is subtracted from `lockedAmount`. However, since `insurances[_id].status` does not get set to `false`, a user can call `unlock()` multiple times for the same id, resulting in `lockedAmount` being way smaller than it should be since `insurances[_id].amount` is subtracted multiple times. 

## Impact
`lockedAmount` is used to calculate the amount of underlying tokens available for withdrawals. If `lockedAmount` is lower than it should be users are able to withdraw more underlying tokens than available for withdrawals.

## Proof of Concept
Typo in `unlock()`:
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L360-L362

Calculation of underlying tokens available for withdrawal:
- https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L836

## Recommended Mitigation Steps
Change `insurances[_id].status == false;` to `insurances[_id].status = false;`

