## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [Missing access control in non-batched InflationManager execute funtions](https://github.com/code-423n4/2022-05-backd-findings/issues/56) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/InflationManager.sol#L145-L155
https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/InflationManager.sol#L236-L249
https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/InflationManager.sol#L321-L330


# Vulnerability details

## Impact

Several actions need to be prepared and go through a time-lock before they can be executed. `InflationManager` allows anyone to call the single action execute function but requires `onlyRoles2(Roles.GOVERNANCE, Roles.INFLATION_MANAGER)` for the batched versions. This looks like an oversight since the same access control level should be enforced.

For example, the `executeLpPoolWeight` function allows anyone to call it:

https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/InflationManager.sol#L241-L249

```
function executeLpPoolWeight(address lpToken) external override returns (uint256) {
   (...)
}
``` 

But the batched version enforces the caller to have `GOVERNANCE` or `INFLATION_MANAGER` roles:

https://github.com/code-423n4/2022-05-backd/blob/main/protocol/contracts/tokenomics/InflationManager.sol#L284-L301

```
function batchExecuteLpPoolWeights(address[] calldata lpTokens)
        external
        override
        onlyRoles2(Roles.GOVERNANCE, Roles.INFLATION_MANAGER)
        returns (bool)
    {
```

The same happens in `executeAmmTokenWeight` versus `batchExecuteAmmTokenWeights` , as well as `executeKeeperPoolWeight` versus `batchExecuteKeeperPoolWeights`.

If only trusted roles should be able to execute pending actions then the `onlyRoles` modifier should be added to the non-batched functions.

Scenarios where you would not want to allow anyone to execute could include potential votes/changes that may trigger a bug or undesired behavior noticed after it had already been approved.

## Tools Used

vim

## Recommended Mitigation Steps

Enforce the proper access control mechanism in non-batched execute functions.

