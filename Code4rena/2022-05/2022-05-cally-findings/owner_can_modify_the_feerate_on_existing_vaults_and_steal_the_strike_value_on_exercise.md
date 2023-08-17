## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Owner can modify the feeRate on existing vaults and steal the strike value on exercise](https://github.com/code-423n4/2022-05-cally-findings/issues/47) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L117-L121


# Vulnerability details

## Impact
Owner can steal the exercise cost which should have gone to the option seller

## Proof of Concept
There are no restrictions on when the owner can set the `feeRate`:
```solidity
File: contracts/src/Cally.sol   #1

117       /// @notice Sets the fee that is applied on exercise
118       /// @param feeRate_ The new fee rate: fee = 1% = (1 / 100) * 1e18
119       function setFee(uint256 feeRate_) external onlyOwner {
120           feeRate = feeRate_;
121       }
```
https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L117-L121

By using a rate that consumes the exercise cost, the owner can steal Ether from the seller:
```solidity
File: contracts/src/Cally.sol   #2

282           uint256 fee = 0;
283           if (feeRate > 0) {
284               fee = (msg.value * feeRate) / 1e18;
285               protocolUnclaimedFees += fee;
286           }
287   
288           // increment vault beneficiary's ETH balance
289           ethBalance[getVaultBeneficiary(vaultId)] += msg.value - fee;
```
https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L282-L289

The owner can wait for a particularly large-value NFT, snipe that one option, then retire

## Tools Used
Code inspection

## Recommended Mitigation Steps
Fix the fee rate per vault during vault creation


