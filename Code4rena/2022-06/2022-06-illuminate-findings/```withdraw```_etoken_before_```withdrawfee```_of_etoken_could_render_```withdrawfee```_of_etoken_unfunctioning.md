## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [```withdraw``` eToken before ```withdrawFee``` of eToken could render ```withdrawFee``` of eToken unfunctioning](https://github.com/code-423n4/2022-06-illuminate-findings/issues/209) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L705-L720
https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L659-L675


# Vulnerability details

```withdrawFee``` of eToken requires the amount of eToken in ```Lender.sol``` >= ```fees[eToken]``` so ```Safe.transfer``` will not revert. However if the admin ```withdraw(eToken)``` first, the balance of eToken in ```Lender.sol``` will equal to zero while ```fees[eToken]``` remains the same and ```withdrawFee(eToken)``` will become unfunctioning since eToken in the contract does not match ```fees[eToken]```. The admin will need to rely on ```withdraw```, which takes 3 days before transfering, to get the future fees of eToken.

### Mitigations
add ```fees[eToken] = 0;``` after ```withdrawals[e] = 0;```  in ```withdraw```. 

