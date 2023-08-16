## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant checks](https://github.com/code-423n4/2021-11-malt-findings/issues/316) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Bonding.sol#L114-L132

```solidity=114
function unbondAndBreak(uint256 amount)
    external
  {
    require(amount > 0, "Cannot unbond 0");

    uint256 bondedBalance = balanceOfBonded(msg.sender);

    require(bondedBalance > 0, "< bonded balance");
    require(amount <= bondedBalance, "< bonded balance");

    // Avoid leaving dust behind
    if (amount.add(1e16) > bondedBalance) {
      amount = bondedBalance;
    }

    miningService.onUnbond(msg.sender, amount);

    _unbondAndBreak(amount);
  }
```

L121, the check of `bondedBalance > 0` is unnecessary, since the L122 already included the same check.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/ERC20Permit.sol#L118-L127

```solidity=118{121-122, 124}
  function transferAndCall(address to, uint value, bytes calldata data) external returns (bool) {
    require(to != address(0) || to != address(this));

    uint256 balance = balanceOf(msg.sender);
    require(balance >= value, "ERC20Permit: transfer amount exceeds balance");

    _transfer(msg.sender, to, value);

    return ITransferReceiver(to).onTokenTransfer(msg.sender, value, data);
  }
```

L121-L122, the check of `balance >= value` is unnecessary, since the L124 already included the same check.

