## Tags

- bug
- documentation
- 2 (Med Risk)
- sponsor confirmed
- valid

# [changeTokenOf makes it impossible for holders of oldToken to redeem the overflowed assets.](https://github.com/code-423n4/2022-07-juicebox-findings/issues/83) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena//blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBController.sol#L588-L606


# Vulnerability details

## Impact
When the owner calls the changeTokenOf function of the JBController contract, the token corresponding to the current project will be changed, which will make the oldToken holder unable to redeem the overflowing assets.
## Proof of Concept
https://github.com/jbx-protocol/juice-contracts-v2-code4rena//blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBController.sol#L588-L606
https://github.com/jbx-protocol/juice-contracts-v2-code4rena//blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBTokenStore.sol#L236-L269
## Tools Used
None
## Recommended Mitigation Steps
Consider adding a delay to changeTokenOf, or adding a function to convert oldToken to newToken

