## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-06

# [StabilizerNode.stabilize uses stale GlobalImpliedCollateralService data, which will make stabilize incorrect](https://github.com/code-423n4/2023-02-malt-findings/issues/9) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/StabilizerNode.sol#L161-L237


# Vulnerability details

## Impact
In StabilizerNode.stabilize, impliedCollateralService.syncGlobalCollateral() is called only at the end of the function to synchronize the GlobalImpliedCollateralService data.
```solidity
    if (!_shouldAdjustSupply(exchangeRate, stabilizeToPeg)) {
      lastStabilize = block.timestamp;
      impliedCollateralService.syncGlobalCollateral();
      return;
    }
...
    if (trackAfterStabilize) {
      maltDataLab.trackPool();
    }
    impliedCollateralService.syncGlobalCollateral();
    lastStabilize = block.timestamp;
  }
```
syncGlobalCollateral will use the data in getCollateralizedMalt(), which includes the collateralToken balance in overflowPool/swingTraderManager/liquidityExtension and the malt balance in swingTraderManager.
```solidity
  function syncGlobalCollateral() public onlyActive {
    globalIC.sync(getCollateralizedMalt());
  }
...
  function getCollateralizedMalt() public view returns (PoolCollateral memory) {
    uint256 target = maltDataLab.priceTarget();

    uint256 unity = 10**collateralToken.decimals();

    // Convert all balances to be denominated in units of Malt target price
    uint256 overflowBalance = maltDataLab.rewardToMaltDecimals((collateralToken.balanceOf(
      address(overflowPool)
    ) * unity) / target);
    uint256 liquidityExtensionBalance = (collateralToken.balanceOf(
      address(liquidityExtension)
    ) * unity) / target;
    (
      uint256 swingTraderMaltBalance,
      uint256 swingTraderBalance
    ) = swingTraderManager.getTokenBalances();
    swingTraderBalance = (swingTraderBalance * unity) / target;
```
Since StabilizerNode.stabilize will use the results of maltDataLab.getActualPriceTarget/getSwingTraderEntryPrice to stabilize, and maltDataLab.getActualPriceTarget/getSwingTraderEntryPrice will use `GlobalImpliedCollateralService.collateralRatio` , to ensure correct stabilization, the data in GlobalServiceImpliedCollateralService should be the latest.
```solidity
  function getActualPriceTarget() external view returns (uint256) {
    uint256 unity = 10**collateralToken.decimals();
    uint256 icTotal = maltToRewardDecimals(globalIC.collateralRatio());
...
  function getSwingTraderEntryPrice()
    external
    view
    returns (uint256 stEntryPrice)
  {
    uint256 unity = 10**collateralToken.decimals();
    uint256 icTotal = maltToRewardDecimals(globalIC.collateralRatio());
```
But since impliedCollateralService.syncGlobalCollateral() is not called before StabilizerNode.stabilize calls maltDataLab.getActualPriceTarget/getSwingTraderEntryPrice, this will cause StabilizerNode.stabilize to use stale GlobalImpliedCollateralService data, which will make stabilize incorrect.

A simple example would be:
1. impliedCollateralService.syncGlobalCollateral() is called to synchronize the latest data
2. SwingTraderManager.delegateCapital is called, and the collateralToken is taken out from SwingTrader, which will make the `GlobalImpliedCollateralService.collateralRatio` larger than the actual collateralRatio.
```solidity
  function delegateCapital(uint256 amount, address destination)
    external
    onlyRoleMalt(CAPITAL_DELEGATE_ROLE, "Must have capital delegation privs")
    onlyActive
  {
    collateralToken.safeTransfer(destination, amount);
    emit Delegation(amount, destination, msg.sender);
  }
...
  function collateralRatio() public view returns (uint256) {
    uint256 decimals = malt.decimals();
    uint256 totalSupply = malt.totalSupply();
    if (totalSupply == 0) {
      return 0;
    }
    return (collateral.total * (10**decimals)) / totalSupply; // @audit: collateral.total is larger than the actual
  }
```
3. When StabilizerNode.stabilize is called, it will use the stale collateralRatio for calculation. If the collateralRatio is too large, the results of maltDataLab.getActualPriceTarget/getSwingTraderEntryPrice will be incorrect, thus making stabilize incorrect.

Since stabilize is a core function of the protocol, stabilizing with the wrong data is likely to cause malt to be depegged, so the vulnerability should be high-risk.

## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/StabilizerNode.sol#L161-L237
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/ImpliedCollateralService.sol#L89-L131
## Tools Used
None
## Recommended Mitigation Steps
Call impliedCollateralService.syncGlobalCollateral() before StabilizerNode.stabilize calls maltDataLab.getActualPriceTarget.

```diff
  function stabilize() external nonReentrant onlyEOA onlyActive whenNotPaused {
    // Ensure data consistency
    maltDataLab.trackPool();

    // Finalize auction if possible before potentially starting a new one
    auction.checkAuctionFinalization();

+  impliedCollateralService.syncGlobalCollateral();

    require(
      block.timestamp >= stabilizeWindowEnd || _stabilityWindowOverride(),
      "Can't call stabilize"
    );
    stabilizeWindowEnd = block.timestamp + stabilizeBackoffPeriod;

    // used in 3 location.
    uint256 exchangeRate = maltDataLab.maltPriceAverage(priceAveragePeriod);
    bool stabilizeToPeg = onlyStabilizeToPeg; // gas

    if (!_shouldAdjustSupply(exchangeRate, stabilizeToPeg)) {
      lastStabilize = block.timestamp;
      impliedCollateralService.syncGlobalCollateral();
      return;
    }

    emit Stabilize(block.timestamp, exchangeRate);

    (uint256 livePrice, ) = dexHandler.maltMarketPrice();

    uint256 priceTarget = maltDataLab.getActualPriceTarget();
```