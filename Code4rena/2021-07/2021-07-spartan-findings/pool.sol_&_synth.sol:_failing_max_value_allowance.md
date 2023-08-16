## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Pool.sol & Synth.sol: Failing Max Value Allowance](https://github.com/code-423n4/2021-07-spartan-findings/issues/29) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In the `_approve` function, if the allowance passed in is `type(uint256).max`, nothing happens (ie. allowance will still remain at previous value). Contract integrations (DEXes for example) tend to hardcode this value to set maximum allowance initially, but this will result in zero allowance given instead.

This also makes the comment `// No need to re-approve if already max` misleading, because the max allowance attainable is `type(uint256).max - 1`, and re-approval does happen in this case.

This affects the `approveAndCall` implementation since it uses `type(uint256).max` as the allowance amount, but the resulting allowance set is zero.

### Recommended Mitigation Steps

Keep it simple, remove the condition.

```jsx
function _approve(address owner, address spender, uint256 amount) internal virtual {
        require(owner != address(0), "!owner");
        require(spender != address(0), "!spender");
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
```

