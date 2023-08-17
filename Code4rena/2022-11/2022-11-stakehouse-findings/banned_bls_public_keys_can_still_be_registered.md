## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-11

# [Banned BLS public keys can still be registered](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/144) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/a0558ed7b12e1ace1fe5c07970c7fc07eb00eebd/contracts/liquid-staking/LiquidStakingManager.sol#L469


# Vulnerability details

## Impact
In `registerBLSPublicKeys`, it should be checked (according to the comment and error) if a BLS public key is part of the LSD network and not banned:
```solidity
// check if the BLS public key is part of LSD network and is not banned
require(isBLSPublicKeyPartOfLSDNetwork(_blsPublicKey) == false, "BLS public key is banned or not a part of LSD network");
```
However, this is not actually checked. The function `isBLSPublicKeyPartOfLSDNetwork` only checks if the public key is part of the LSD network:
```solidity
function isBLSPublicKeyPartOfLSDNetwork(bytes calldata _blsPublicKeyOfKnot) public virtual view returns (bool) {
        return smartWalletOfKnot[_blsPublicKeyOfKnot] != address(0);
}
```
The function `isBLSPublicKeyBanned` would perform both checks and should be called here:
```solidity
function isBLSPublicKeyBanned(bytes calldata _blsPublicKeyOfKnot) public virtual view returns (bool) {
        return !isBLSPublicKeyPartOfLSDNetwork(_blsPublicKeyOfKnot) || bannedBLSPublicKeys[_blsPublicKeyOfKnot] != address(0);
}
```

Because of that, it is possible to pass banned BLS public keys to `registerBLSPublicKeys` and the call will succeed.

## Recommended Mitigation Steps
Use `isBLSPublicKeyBanned` instead of `isBLSPublicKeyPartOfLSDNetwork`.