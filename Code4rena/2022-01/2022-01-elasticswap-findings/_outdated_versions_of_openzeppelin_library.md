## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ Outdated versions of OpenZeppelin library](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/155) 

# Handle

WatchPug


# Vulnerability details

Outdated versions of OpenZeppelin library are used.

New versions of OpenZeppelin libraries can be more gas efficient. 

For example:

`ERC20.sol` in @openzeppelin/contracts@4.1.0:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.1.0/contracts/token/ERC20/ERC20.sol#L152-L153

```solidity
require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
_approve(sender, _msgSender(), currentAllowance - amount);
```

A gas optimization upgrade has been added to @openzeppelin/contracts@4.4.2:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.4.2/contracts/token/ERC20/ERC20.sol#L158-L161
```solidity
require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
unchecked {
    _approve(sender, _msgSender(), currentAllowance - amount);
}
```

