## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [Voter in VoteEscrowCore can permanently lock user tokens](https://github.com/code-423n4/2022-07-golom-findings/issues/712) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L873-L876
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L883-L886
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L894
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L538
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L1008


# Vulnerability details

## Impact
A malicious voter can arbitrarily increase the number of `attachments` or set the `voted` status of a token to true. This prevents the token from being withdrawn, merged or transfered thereby locking the tokens into the contract for as long as the voter would like. 

I submitted this is as a medium severity because it has external circumstances (a malicious voter) however has a very high impact if it does occur.

## Proof of Concept
1. A user creates a lock for their token and deposits it into the VoteEscrowDelegate/Core contract.
2. The malicious voter then calls either `voting()` or `attach()` thereby preventing the user withdrawing their token after the locked time bypasses

## Tools Used
VS Code
## Recommended Mitigation Steps
I have not seen any use of `voting()` or `attach()` in any of the other contracts so it may be sensible to remove those functions altogether. On the other hand, setting voter to be smart contract which is not malicious offsets this problem.