## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G1] `InsureDAOERC20#transferFrom()` Do not reduce approval on transferFrom if current allowance is type(uint256).max](https://github.com/code-423n4/2022-01-insure-findings/issues/213) 

# Handle

WatchPug


# Vulnerability details

The Wrapped Ether (WETH) ERC-20 contract has a gas optimization that does not update the allowance if it is the max uint.

The latest version of OpenZeppelin's ERC20 token contract also adopted this optimization.

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
-   https://github.com/OpenZeppelin/openzeppelin-contracts/blob/80d8da05644ceef3cd8e81860882571f037f8667/contracts/token/ERC20/ERC20.sol#L162
-   https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3085

### Recommendation

Change to:

```solidity
function transferFrom(
    address sender,
    address recipient,
    uint256 amount
) public virtual override returns (bool) {
    _transfer(sender, recipient, amount);

    uint256 currentAllowance = _allowances[sender][_msgSender()];
    if (currentAllowance != type(uint256).max) {
        require(
            currentAllowance >= amount,
            "ERC20: transfer amount exceeds allowance"
        );

        _approve(sender, _msgSender(), currentAllowance - amount);
    }

    return true;
}
```

