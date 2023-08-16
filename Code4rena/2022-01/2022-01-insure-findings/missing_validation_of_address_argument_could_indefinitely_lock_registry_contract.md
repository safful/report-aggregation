## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Missing validation of address argument could indefinitely lock Registry contract](https://github.com/code-423n4/2022-01-insure-findings/issues/110) 

# Handle

defsec


# Vulnerability details

## Impact

the owner parameter are used for the onlyOwner modifier. In the state variable , proper check up should be done , other wise error in these state variable can lead to redeployment of contract. If the zero address is assigned to rebalanceManager parameter, that will fail all Owner functions.

## Proof of Concept

1. Navigate to the following contract functions.

"https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Registry.sol#L31"

2. Adding zero address into the owner leads to failure of onlyOwner only functions.

## Tools Used

Code Review

## Recommended Mitigation Steps

Add proper zero address validation.

