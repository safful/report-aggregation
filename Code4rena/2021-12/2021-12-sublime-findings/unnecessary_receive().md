## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unnecessary receive()](https://github.com/code-423n4/2021-12-sublime-findings/issues/99) 

# Handle

Jujic


# Vulnerability details

## Impact
There doesn't seem to be a use case for the existence of the `receive()` function. In fact, I will recommend removing it as it will prevent accidental native token transfers to the contract.

## Proof of Concept
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/SavingsAccount/SavingsAccount.sol#L481

## Tools Used
VSC
## Recommended Mitigation Steps
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/SavingsAccount/SavingsAccount.sol#L481

