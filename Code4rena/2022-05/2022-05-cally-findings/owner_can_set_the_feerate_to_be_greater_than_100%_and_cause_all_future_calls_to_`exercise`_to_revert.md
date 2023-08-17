## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Owner can set the feeRate to be greater than 100% and cause all future calls to `exercise` to revert](https://github.com/code-423n4/2022-05-cally-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L288-L289


# Vulnerability details

## Impact
The owner can force options to be non-exercisable, collecting premium without risking the loss of their NFT/tokens

## Proof of Concept
After a buyer buys an option owned by the owner, the owner can change the fee rate to be close to `type(uint256).max`, which will cause the subtraction below to always underflow, preventing the exercise of the option. Once the option expires, the owner can change the fee back and wait for another buyer

```solidity
File: contracts/src/Cally.sol   #1

288           // increment vault beneficiary's ETH balance
289           ethBalance[getVaultBeneficiary(vaultId)] += msg.value - fee;
```
https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L288-L289

## Tools Used
Code inspection

## Recommended Mitigation Steps
Add reasonable fee rate bounds checks in the `setFee()` function


