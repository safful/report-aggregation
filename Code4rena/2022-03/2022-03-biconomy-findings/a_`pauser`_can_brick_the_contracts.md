## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [A `pauser` can brick the contracts](https://github.com/code-423n4/2022-03-biconomy-findings/issues/137) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/security/Pausable.sol#L65-L68


# Vulnerability details

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/security/Pausable.sol#L65-L68

```solidity
    function renouncePauser() external virtual onlyPauser {
        emit PauserChanged(_pauser, address(0));
        _pauser = address(0);
    }
```

A malicious or compromised `pauser` can call `pause()` and `renouncePauser()` to brick the contract and all the funds can be frozen.

### PoC

Given:

* Alice (EOA) is the `pauser` of the contract.

1. Alice calls `pause()` ;
2. Alice calls `renouncePauser()`;


As a result, most of the contract's methods are now unavailable, and this cannot be reversed even by the `owner`.

### Recommendation

Consider removing `renouncePauser()`, or requiring the contract not in `paused` mode when `renouncePauser()`.

