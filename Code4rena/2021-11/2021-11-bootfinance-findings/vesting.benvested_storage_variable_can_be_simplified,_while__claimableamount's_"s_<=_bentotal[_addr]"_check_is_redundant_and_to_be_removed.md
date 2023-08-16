## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Vesting.benVested storage variable can be simplified, while _claimableAmount's "s <= benTotal[_addr]" check is redundant and to be removed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/186) 

# Handle

hyh


# Vulnerability details

## Proof of Concept

The only usage is in _claimableAmount function and can be rewritten with one uint256 storage variable.
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L183
benVested cannot be used to get current state as it is updated only during claim() and revoke() calls and calcClaimableAmount() to be used instead.

The timelocks totals and benTotal cannot differ as timelocks are updated and deleted in vest() and revoke() functions only correspondingly, while there benTotal is updated with very same amount without any additional conditions.
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L91
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L128

This way the 's <= benTotal[_addr]' check is redundant and to be removed.

## Impact

's <= benTotal[_addr]' check can be dangerous: the totals and benTotal cannot differ, while if there would be such a possibility, various attacks might be possible, for example a griefing one, when claim() always fails because of this check, and so on.

I.e. now benVested can be simplified and check is not needed, while if there would be such a situation when it is needed, simple check as it is cannot be sufficient, and some code redesign should be done instead.

## Recommended Mitigation Steps

Code update:
Now:
mapping(address => uint256[2]) public benVested;
...
uint256 completely_vested = 0;
uint256 partial_sum = 0;
...
completely_vested = completely_vested.add(timelocks[_addr][i].amount);
...
partial_sum = partial_sum.add(claimable);
...
benVested[_addr][0] = benVested[_addr][0].add(completely_vested);
benVested[_addr][1] = partial_sum;
uint256 s = benVested[_addr][0].add(partial_sum);
assert(s <= benTotal[_addr]);
return s;

To be:
mapping(address => uint256) public benVested;
...
uint256 currently_vested = 0;
...
currently_vested = currently_vested.add(timelocks[_addr][i].amount);
...
currently_vested = currently_vested.add(claimable);
...
uint256 s = benVested[_addr].add(currently_vested);
benVested[_addr] = s;
return s;

Also, cleaning in revoke() simplifies to
benVested[_addr] = 0;
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L127

