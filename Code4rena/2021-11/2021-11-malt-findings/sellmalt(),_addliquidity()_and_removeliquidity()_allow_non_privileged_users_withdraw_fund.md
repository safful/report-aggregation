## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [sellMalt(), addLiquidity() and removeLiquidity() Allow Non Privileged Users Withdraw Fund](https://github.com/code-423n4/2021-11-malt-findings/issues/120) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
Is Not Uncommon Normal Users Accidentally Send Tokens into Contract.

ENS Airdrop is a Good Example Normal Users Accidentally Send Tokens into Contract:
https://discuss.ens.domains/t/social-amend-airdrop-proposal-to-include-accidentally-returned-funds/6975

In UniswapHandler.sol, sellMalt(), addLiquidity() and removeLiquidity() Have No Access Control. When Normal Users Accidently Deposit Tokens into the Contract, Any Random Persons/Bot Can Withdraw the Tokens because it will safeTransfer to msg.sender who find out there is token balance in the contract.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L185-L219
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L221-L245

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add relevant access control, probably Only StabilizerNode and Admin have Access to this contract various functions like sellMalt(), addLiquidity() and removeLiquidity() etc.


