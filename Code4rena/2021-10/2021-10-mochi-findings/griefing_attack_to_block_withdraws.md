## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [griefing attack to block withdraws](https://github.com/code-423n4/2021-10-mochi-findings/issues/21) 

# Handle

gpersoon


# Vulnerability details

## Impact
Every time you deposit some assets in the vault (via deposit() of MochiVault.sol) then "lastDeposit[_id]" is set to block.timestamp.
The modifier wait() checks this value and makes sure you cannot withdraw for "delay()" blocks.
The default value for delay() is 3 minutes.

Knowing this delay you can do a griefing attack:
On chains with low gas fees: every 3 minutes deposit a tiny amount for a specific NFT-id (which has a large amount of assets).
On chains with high gas fees: monitor the mempool for a withdraw() transaction and frontrun it with a deposit()

This way the owner of the NFT-id can never withdraw the funds.

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/vault/MochiVault.sol#L47-L54

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/vault/MochiVault.sol#L171

https://github.com/code-423n4/2021-10-mochi/blob/806ebf2a364c01ff54d546b07d1bdb0e928f42c6/projects/mochi-core/contracts/profile/MochiProfileV0.sol#L33

## Tools Used

## Recommended Mitigation Steps
Create a mechanism where you only block the withdraw of recently deposited funds


