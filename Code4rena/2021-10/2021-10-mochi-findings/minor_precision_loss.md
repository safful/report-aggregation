## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Minor precision loss](https://github.com/code-423n4/2021-10-mochi-findings/issues/105) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L67-L68

```solidity=67
mochi.transfer(msg.sender, _amount / 2);
mochi.transfer(address(vMochi), _amount / 2);
```

Change to:

```solidity=67
mochi.transfer(msg.sender, _amount / 2);
mochi.transfer(address(vMochi), _amount - _amount / 2);
```

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/treasury/MochiTreasuryV0.sol#L59-L65

```solidity=63
operationShare += updatedFee / 2;
veCRVShare += updatedFee / 2;
```

Change to:

```solidity=63
operationShare += updatedFee / 2;
veCRVShare += updatedFee - updatedFee / 2;
```

