## Tags

- bug
- 3 (High Risk)
- sponsor acknowledged
- sponsor confirmed

# [No safety check in addToken](https://github.com/code-423n4/2021-09-yaxis-findings/issues/3) 

# Handle

jonah1005


# Vulnerability details

## Impact
There's no safety check in `Manager.sol` `addToken`. There are two possible cases that might happen.
1. One token being added twice in a Vault. Token would be counted doubly in the vault. Ref: [Vault.sol#L293-L303](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/Vault.sol#L293-L303). There would be two item in the array when querying `manager.getTokens(address(this));`.

2. A token first being added to two vaults. The value calculation of the first vault would be broken. As `vaults[_token] = _vault;` would point to the other vault.

Permission keys should always be treated cautiously. However, calling the same initialize function twice should not be able to destroy the vault. Also, as the protocol develops, there's likely that one token is supported in two vaults. The DAO may mistakenly add the same token twice. I consider this a high-risk issue.


## Proof of Concept
Adding same token twice would not raise any error here.
```
manager.functions.addToken(vault.address, dai.address).transact()
manager.functions.addToken(vault.address, dai.address).transact()
```

## Tools Used
Hardhat

## Recommended Mitigation Steps
I recommend to add two checks
```solidity
require(vaults[_token] == address(0));
bool notFound = True;
for(uint256 i; i < tokens[_vault].length; i++) {
    if (tokens[_vault] == _token) {
        notFound = False;
    }
}
require(notFound, "duplicate token");
```

