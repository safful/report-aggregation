## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Allowance checks not correctly implemented](https://github.com/code-423n4/2021-08-notional-findings/issues/66) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `nTokenAction` implements two token approvals, the `nTokenWhitelist` which is always used first, and the `nTokenAllowance` which is checked second.
If the `nTokenWhitelist` does _not_ have enough allowance for the transfer, the transaction fails, even in the case where `nTokenAllowance` still has enough allowance.

## Impact
Transfers that have sufficient allowance fail in certain cases.

## Recommended Mitigation Steps
Instead of reverting if the `nTokenWhitelist` allowance is not enough, default to the `nTokenAllowance` case:

```solidity
// something like this

uint256 requiredAllowance = amount;

uint256 allowance = nTokenWhitelist[from][spender];
// use whitelist allowance first
if (allowance > 0) {
    uint256 min = amount < allowance ? amount : allowance;
    requiredAllowance -= min;
    allowance = allowance.sub(min);
    nTokenWhitelist[from][spender] = allowance;
}

// use currency-specific allowance now
if(requiredAllowance > 0)
    // This is the specific allowance for the nToken.
    allowance = nTokenAllowance[from][spender][currencyId];
    require(allowance >= requiredAllowance, "Insufficient allowance");
    allowance = allowance.sub(requiredAllowance);
    nTokenAllowance[from][spender][currencyId] = allowance;
}
```


