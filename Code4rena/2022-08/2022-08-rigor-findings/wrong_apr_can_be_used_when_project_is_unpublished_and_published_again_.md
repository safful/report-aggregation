## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- edited-by-warden
- valid

# [Wrong APR can be used when project is unpublished and published again ](https://github.com/code-423n4/2022-08-rigor-findings/issues/83) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/e35f5f61be9ff4b8dc5153e313419ac42964d1fd/contracts/Community.sol#L267


# Vulnerability details

## Impact
When a project is unpublished from a community, it can still owe money to this community (on which it needs to pay interest according to the specified APR). However, when the project is later published again in this community, the APR can be overwritten and the overwritten APR is used for the calculation of the interest for the old project (when it was unpublished).

## Proof Of Concept
1.) Project A is published in community I with an APR of 3%. The community lends 1,000,000 USD to the project.
2.) Project A is unpublished, the `lentAmount` is still 1,000,000 USD.
3.) During one year, no calls to `repayLender`, `reduceDebt`, or `escrow` happens, i.e. the interest is never added and the `lastTimestamp` not updated.
4.) After one year, the project is published again in the same community. Because the FED raised interest rates, it is specified that the APR should be 5% from now on.
5.) Another $1,000,000 is lent to the project by calling `lendToProject`. Now, `claimInterest` is called which calculates the interest of the last year for the first million. However, the function already uses the new APR of 5%, meaning the added interest is 50,000 USD instead of the correct 30,000 USD. 

## Recommended Mitigation Steps
When publishing a project, if the `lentAmount` for the community is non-zero, calculate the interest before updating the APR.