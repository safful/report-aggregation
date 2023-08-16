## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Race condition on ERC20 approval](https://github.com/code-423n4/2021-11-malt-findings/issues/276) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L112-L116

```solidity=112
  function approveAndCall(address spender, uint256 value, bytes calldata data) external returns (bool) {
    _approve(msg.sender, spender, value);

    return IApprovalReceiver(spender).onTokenApproval(msg.sender, value, data);
  }
```

```solidity=208
function _transfer(address sender, address recipient, uint256 amount) internal virtual {
    require(sender != address(0), "ERC20: transfer from the zero address");
    require(recipient != address(0), "ERC20: transfer to the zero address");

    _beforeTokenTransfer(sender, recipient, amount);

    _balances[sender] = _balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");
    _balances[recipient] = _balances[recipient].add(amount);
    emit Transfer(sender, recipient, amount);
}
```

Using approve() to manage allowances opens yourself and users of the token up to frontrunning.
Best practice, but doesn't usually matter.

[Explanation](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit) of this possible attack vector

See also: [0xProject/0x-monorepo#850](https://github.com/0xProject/0x-monorepo/issues/850)

Using increase/decreaseAllowance instead is recommended.

