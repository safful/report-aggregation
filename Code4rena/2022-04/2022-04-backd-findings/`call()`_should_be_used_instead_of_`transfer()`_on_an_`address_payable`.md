## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [`call()` should be used instead of `transfer()` on an `address payable`](https://github.com/code-423n4/2022-04-backd-findings/issues/52) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/actions/topup/TopUpAction.sol#L291
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/pool/EthPool.sol#L30
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/strategies/BkdEthCvx.sol#L77
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/strategies/BkdEthCvx.sol#L93
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/strategies/BkdEthCvx.sol#L117
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/vault/EthVault.sol#L29
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/vault/EthVault.sol#L37
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/vault/VaultReserve.sol#L81


# Vulnerability details

This is a classic Code4rena issue: 
- https://github.com/code-423n4/2021-04-meebits-findings/issues/2
- https://github.com/code-423n4/2021-10-tally-findings/issues/20
- https://github.com/code-423n4/2022-01-openleverage-findings/issues/75

## Impact
The use of the deprecated `transfer()` function for an address will inevitably make the transaction fail when:

1. The claimer smart contract does not implement a payable function.
2. The claimer smart contract does implement a payable fallback which uses more than 2300 gas unit.
3. The claimer smart contract implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.

Additionally, using higher than 2300 gas might be mandatory for some multisig wallets.

## Impacted lines:

```solidity
backd/contracts/pool/EthPool.sol:
  30:         to.transfer(amount);

backd/contracts/strategies/BkdEthCvx.sol:
   77:             payable(vault).transfer(amount);
   93:         payable(vault).transfer(amount);
  117:         payable(vault).transfer(underlyingBalance);

backd/contracts/vault/EthVault.sol:
  29:         payable(to).transfer(amount); 
  37:         payable(addressProvider.getTreasury()).transfer(amount);  

backd/contracts/vault/VaultReserve.sol:
  81:             payable(msg.sender).transfer(amount);
```

## Recommended Mitigation 
I recommend using `call()` instead of `transfer()`

