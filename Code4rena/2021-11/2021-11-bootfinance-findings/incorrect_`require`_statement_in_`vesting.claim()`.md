## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect `require` Statement in `Vesting.claim()`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/306) 

# Handle

leastwood


# Vulnerability details

## Impact

The `claim()` function asserts that the claimable amount is strictly less than `benTotal` for a given user. However, this does not take into account previously claimed tokens, hence the `require` does not accurately depict its intended behaviour.

## Proof of Concept

https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L197

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider updating this `require` statement to account for already claimed tokens. This could look like the following:
`require(amount.add(benClaimed[msg.sender]) <= benTotal[msg.sender], "Cannot withdraw more than total vested amount");`

