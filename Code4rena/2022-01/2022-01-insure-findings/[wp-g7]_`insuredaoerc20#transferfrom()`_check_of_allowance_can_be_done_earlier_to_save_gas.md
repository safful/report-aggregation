## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G7] `InsureDAOERC20#transferFrom()` Check of allowance can be done earlier to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/219) 

# Handle

WatchPug


# Vulnerability details

Check of allowance can be done earlier (before `_transfer()`) to save some gas for failure transactions.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/InsureDAOERC20.sol#L152-L168

```solidity
function transferFrom(
    address sender,
    address recipient,
    uint256 amount
) public virtual override returns (bool) {
    _transfer(sender, recipient, amount);

    uint256 currentAllowance = _allowances[sender][_msgSender()];
    require(
        currentAllowance >= amount,
        "ERC20: transfer amount exceeds allowance"
    );

    _approve(sender, _msgSender(), currentAllowance - amount);

    return true;
}
```


See:
-   https://github.com/OpenZeppelin/openzeppelin-contracts/blob/80d8da05644ceef3cd8e81860882571f037f8667/contracts/token/ERC20/ERC20.sol#L162-L169


