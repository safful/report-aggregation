## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Potential DoS when removing keeper gauge](https://github.com/code-423n4/2022-05-backd-findings/issues/71) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/InflationManager.sol#L609-L618
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/KeeperGauge.sol#L82
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/actions/topup/TopUpActionFeeHandler.sol#L95-L98
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/actions/topup/TopUpAction.sol#L807
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/actions/topup/TopUpAction.sol#L653


# Vulnerability details

## Impact
When `_removeKeeperGauge` is called, there is no guarantee that the keeper gauge isn't currently in use by any `TopUpActionFeeHandler`. If it's still in use, any top up action executions will be disabled as reporting fees in `KeeperGauge.sol` will revert:
```
function reportFees(
    address beneficiary,
    uint256 amount,
    address lpTokenAddress
) external override returns (bool) {
    ...
    require(!killed, Error.CONTRACT_PAUSED); // gauge is killed by InflationManager
    ...
    return true;
}
```
If this happened during extreme market movements, some positions that require a top up will not be executed and be in risk of being liquidated.

## Proof of Concept
- Alice registers a top up action.
- Governance calls `InflationManager.removeKeeperGauge`, replacing an old keeper gauge. However, governance forgot to call `TopUpActionFeeHandler.prepareKeeperGauge` so `TopUpActionFeeHandler.getKeeperGauge` still points to the killed gauge.
- Market moved and Alice's position should now be executed by keepers, however any attempt to execute will revert:
    ```
    > Keeper calls TopUpAction.execute();
    > _payFees();
    > IActionFeeHandler(feeHandler).payFees();
    > IKeeperGauge(keeperGauge).reportFees();
    > reverts as gauge is already killed
    ```
- Governance noticed and calls `prepareKeeperGauge`  with a 3 days delay.
- Alice's position got liquidated before the change is executed.

## Recommended Mitigation Steps
Consider adding an on-chain check to ensure that the keeper gauge is not in use before removing them.


