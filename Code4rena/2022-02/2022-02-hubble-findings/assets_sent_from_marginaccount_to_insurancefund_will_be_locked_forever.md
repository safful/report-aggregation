## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Assets sent from MarginAccount to InsuranceFund will be locked forever](https://github.com/code-423n4/2022-02-hubble-findings/issues/128) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/ed1d885d5dbc2eae24e43c3ecbf291a0f5a52765/contracts/MarginAccount.sol#L377


# Vulnerability details

# Impact
Assets sent from MarginAccount to InsuranceFund will be locked forever

# Proof of Concept
The insurance fund doesn't have a way to transfer non-vusd out of the contract.

Assets transferred to the InsuranceFund will be locked forever.

# Mitigation
Have a way for governance to sweep tokens to swap them.

