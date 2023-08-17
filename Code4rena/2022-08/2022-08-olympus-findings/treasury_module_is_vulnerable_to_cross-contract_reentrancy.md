## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Treasury module is vulnerable to cross-contract reentrancy](https://github.com/code-423n4/2022-08-olympus-findings/issues/426) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L108-L112


# Vulnerability details

## Impact
An attacker can pay back their loan to the treasury module with protocol-owned tokens. This will cause their loan to decrease despite the protocol won't be given funds for it.

## Proof of Concept
The code first measures the number of tokens in the treasury, then transfers an amount to the contract and checks the change it caused. This is put behind a nonReentrant modifier so that one can't use the same balance change to pay back multiple parts of (potentially) multiple loans.

The problem arises when the treasury doesn't only claim tokens from paying back loans, but also claims protocol revenue. Since, an attacker can gain execution in the moment the funds are pulled to the treasury to trigger any function that grants treasury this type of tokens (collects protocol revenue). The contract will count these tokens as paying back one's loan since this happened between balance measurements.

## Recommended Mitigation Steps
Add a function used to pull a token to the contract and mark it nonReentrant. Any transfer of tokens to the treasury should be done through that function.