## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`OverlayToken.sol` Check of allowance can be done earlier to save gas](https://github.com/code-423n4/2021-11-overlay-findings/issues/66) 

# Handle

WatchPug


# Vulnerability details

Check of allowance can be done earlier to save some gas for failure transactions.

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/ovl/OverlayToken.sol#L118-L137

```solidity=119
function transferFrom(
    address sender,
    address recipient,
    uint256 amount
) public virtual override returns (
    bool success_
) {

    _transfer(sender, recipient, amount);

    uint256 currentAllowance = _allowances[sender][_msgSender()];

    require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");

    unchecked { _approve(sender, _msgSender(), currentAllowance - amount); }

    success_ = true;

}
```

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/ovl/OverlayToken.sol#L167-L186
```solidity=167
function transferFromBurn(
    address sender,
    address recipient,
    uint256 amount,
    uint256 burnt
) public override onlyBurner returns (
    bool success
) {

    _transferBurn(sender, recipient, amount, burnt);

    uint256 currentAllowance = _allowances[sender][msg.sender];

    require(currentAllowance >= amount + burnt, "OVL:allowance<amount+burnt");

    unchecked { _approve(sender, msg.sender, currentAllowance - amount - burnt); }

    success = true;

}
```

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/ovl/OverlayToken.sol#L241-L260
```solidity=241
function transferFromMint(
    address sender,
    address recipient,
    uint256 amount,
    uint256 minted
) public override onlyMinter returns (
    bool
) {

    _transferMint(sender, recipient, amount, minted);

    uint256 currentAllowance = _allowances[sender][msg.sender];

    require(currentAllowance >= amount, "OVL:allowance<amount");

    unchecked { _approve(sender, msg.sender, currentAllowance - amount); }

    return true;

}
```

