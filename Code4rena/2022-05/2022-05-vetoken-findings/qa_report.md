## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-05-vetoken-findings/issues/254) 

## Low
### Use a 2 step procedure to update feeManager
Use a 2 step procedure to update feeManager to reduce risk of burning the admin key
https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeAssetDepositor.sol#L53-L57

```solidity
    function setFeeManager(address _feeManager) external {
        require(msg.sender == feeManager, "!auth");
        feeManager = _feeManager;
        emit FeeManagerUpdated(_feeManager);
    }
```

## Non Critical
### Outdated Solidity
Consider upgrade to latest 0.8.14