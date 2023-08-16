## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [anyone can create a vault by directly calling the factory](https://github.com/code-423n4/2021-10-mochi-findings/issues/80) 

# Handle

jonah1005


# Vulnerability details


## Impact
[MochiVaultFactory.sol#L26-L37](https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVaultFactory.sol#L26-L37)
There's no permission control in the vaultFactory. Anyone can create a vault. The transaction would be reverted when the government tries to deploy such an asset.

As the protocol checks whether the vault is a valid vault by comparing the contract's address with the computed address, the protocol would recognize the random vault as a valid one. 


I consider this is a medium-risk issue.

## Proof of Concept

Here's a web3.py script to trigger the bug.
```py
vault_factory.functions.deployVault(usdt.address).transact()
## this tx would be reverted
profile.functions.registerAssetByGov([usdt.address], [3]).transact()
```

## Tools Used

None

## Recommended Mitigation Steps
Recommend to add a check.
```solidity
require(msg.sender == engine, "!engine");
```

