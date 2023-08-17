## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Automation / management can be set for not yet existing vault](https://github.com/code-423n4/2022-08-mimo-findings/issues/68) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/9adf46f2efc61898247c719f2f948b41d5d62bbe/contracts/actions/automated/MIMOAutoAction.sol#L33
https://github.com/code-423n4/2022-08-mimo/blob/9adf46f2efc61898247c719f2f948b41d5d62bbe/contracts/actions/managed/MIMOManagedAction.sol#L35


# Vulnerability details

## Impact & Proof Of Concept
`vaultOwner` returns zero for a non-existing `vaultId`. Similarly, `proxyRegistry.getCurrentProxy(msg.sender)` returns zero when `msg.sender` has not deployed a proxy yet. Those two facts can be combined to set automation for a vault ID that does not exist yet. When this is done by a user without a proxy, it will succeed, as both `vaultOwner` and `mimoProxy` are `address(0)`, i.e. we have `vaultOwner == mimoProxy`.

The consequences of this are quite severe. As soon as the vault is created, it will be an automated vault (with potentially very high fees). An attacker can exploit this by setting very high fees before the creation of the vault and then performing actions for the automated vault, which leads to a loss of funds for the user.

The same attack is possible for `setManagement`.

## Recommended Mitigation Steps
Do not allow setting automation parameters for non-existing vaults, i.e. check that `vaultOwner != address(0)`.