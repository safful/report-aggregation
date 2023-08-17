## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [The noContract modifier does not work as expected.](https://github.com/code-423n4/2022-04-jpegd-findings/issues/11) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/59e288c27e1ff1b47505fea2e5434a7577d85576/contracts/vaults/yVault/yVault.sol#L61
https://github.com/code-423n4/2022-04-jpegd/blob/59e288c27e1ff1b47505fea2e5434a7577d85576/contracts/farming/yVaultLPFarming.sol#L54
https://github.com/code-423n4/2022-04-jpegd/blob/59e288c27e1ff1b47505fea2e5434a7577d85576/contracts/vaults/yVault/yVault.sol#L61


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

The expectation of the noContract modifier is to allow access only to accounts inside EOA or Whitelist, if access is controlled using ! access control with _account.isContract(), then because isContract() gets the size of the code length of the account in question by relying on extcodesize/address.code.length, this means that the restriction can be bypassed when deploying a smart contract through the smart contract's constructor call.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used

## Recommended Mitigation Steps

Modify the code to `require(msg.sender == tx.origin);`

