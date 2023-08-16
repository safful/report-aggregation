## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`FeePoolV0.sol` Lack of input validation](https://github.com/code-423n4/2021-10-mochi-findings/issues/106) 

# Handle

WatchPug


# Vulnerability details

`treasuryRatio` and `vMochiRatio` must be `<= 1e18` to make sure the contract works correctly. Therefore, the input should be checked in the setters.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/feePool/FeePoolV0.sol#L45-L53

```solidity=45
function changeTreasuryRatio(uint256 _ratio) external {
    require(msg.sender == engine.governance(), "!gov");
    treasuryRatio = _ratio;
}

function changevMochiRatio(uint256 _ratio) external {
    require(msg.sender == engine.governance(), "!gov");
    vMochiRatio = _ratio;
}
```

### Recommendation

Change to:

```solidity=45
function changeTreasuryRatio(uint256 _ratio) external {
    require(msg.sender == engine.governance(), "!gov");
    require(_ratio <= 1e18, ">1e18");
    treasuryRatio = _ratio;
}

function changevMochiRatio(uint256 _ratio) external {
    require(msg.sender == engine.governance(), "!gov");
    require(_ratio <= 1e18, ">1e18");
    vMochiRatio = _ratio;
}
```

