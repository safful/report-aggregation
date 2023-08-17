## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-04

# [SwingTraderManager.addSwingTrader will push traderId with active = false to activeTraders](https://github.com/code-423n4/2023-02-malt-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/SwingTraderManager.sol#L397-L447


# Vulnerability details

## Impact
In SwingTraderManager.addSwingTrader, if active = false, the traderId is also pushed to activeTraders.
```solidity
  function addSwingTrader(
    uint256 traderId,
    address _swingTrader,
    bool active,
    string calldata name
  ) external onlyRoleMalt(ADMIN_ROLE, "Must have admin privs") {
    SwingTraderData storage trader = swingTraders[traderId];
    require(traderId > 2 && trader.id == 0, "TraderId already used");
    require(_swingTrader != address(0), "addr(0)");

    swingTraders[traderId] = SwingTraderData({
      id: traderId,
      index: activeTraders.length,
      traderContract: _swingTrader,
      name: name,
      active: active
    });

    activeTraders.push(traderId);

    emit AddSwingTrader(traderId, name, active, _swingTrader);
  }
```
Afterwards, if toggleTraderActive is called on the traderId, the traderId will be pushed to activeTraders again.
```solidity
  function toggleTraderActive(uint256 traderId)
    external
    onlyRoleMalt(ADMIN_ROLE, "Must have admin privs")
  {
    SwingTraderData storage trader = swingTraders[traderId];
    require(trader.id == traderId, "Unknown trader");

    bool active = !trader.active;
    trader.active = active;

    if (active) {
      // setting it to active so add to activeTraders
      trader.index = activeTraders.length;
      activeTraders.push(traderId);
    } else {
```
This means that in getTokenBalances()/calculateSwingTraderMaltRatio(), since there are two identical traderIds in activeTraders, the data in this trader will be calculated twice.
Wrong getTokenBalances() will result in wrong data when syncGlobalCollateral(). 
```solidity
  function getTokenBalances()
    external
    view
    returns (uint256 maltBalance, uint256 collateralBalance)
  {
    uint256[] memory traderIds = activeTraders;
    uint256 length = traderIds.length;

    for (uint256 i; i < length; ++i) {
      SwingTraderData memory trader = swingTraders[activeTraders[i]];
      maltBalance += malt.balanceOf(trader.traderContract);
      collateralBalance += collateralToken.balanceOf(trader.traderContract);
    }
  }
```
Wrong calculateSwingTraderMaltRatio() will cause MaltDataLab.getRealBurnBudget()/getSwingTraderEntryPrice() to be wrong.
```solidity
  function calculateSwingTraderMaltRatio()
    public
    view
    returns (uint256 maltRatio)
  {
    uint256[] memory traderIds = activeTraders;
    uint256 length = traderIds.length;
    uint256 decimals = collateralToken.decimals();
    uint256 maltDecimals = malt.decimals();
    uint256 totalMaltBalance;
    uint256 totalCollateralBalance;

    for (uint256 i; i < length; ++i) {
      SwingTraderData memory trader = swingTraders[activeTraders[i]];
      totalMaltBalance += malt.balanceOf(trader.traderContract);
      totalCollateralBalance += collateralToken.balanceOf(
        trader.traderContract
      );
    }

    totalMaltBalance = maltDataLab.maltToRewardDecimals(totalMaltBalance);

    uint256 stMaltValue = ((totalMaltBalance * maltDataLab.priceTarget()) /
      (10**decimals));

    uint256 netBalance = totalCollateralBalance + stMaltValue;

    if (netBalance > 0) {
      maltRatio = ((stMaltValue * (10**decimals)) / netBalance);
    } else {
      maltRatio = 0;
    }
  }
```
What's more serious is that even if toggleTraderActive is called again, only one traderId will pop up from activeTraders, and the other traderId cannot be popped up.
```solidity
    } else {
      // Becoming inactive so remove from activePools
      uint256 index = trader.index;
      uint256 lastTrader = activeTraders[activeTraders.length - 1];

      activeTraders[index] = lastTrader;
      activeTraders.pop();

      swingTraders[lastTrader].index = index;
      trader.index = 0;
    }
```
This causes the trade to participate in the calculation of getTokenBalances()/calculateSwingTraderMaltRatio() even if the trade is deactive.
Considering that the active parameter is likely to be false when addSwingTrader is called and cannot be recovered, this vulnerability should be high risk
## Proof of Concept
```solidity
  function testAddSwingTrader(address newSwingTrader) public {
    _setupContract();
    vm.assume(newSwingTrader != address(0));
    vm.prank(admin);
    swingTraderManager.addSwingTrader(3, newSwingTrader, false, "Test");

    (
      uint256 id,
      uint256 index,
      address traderContract,
      string memory name,
      bool active
    ) = swingTraderManager.swingTraders(3);

    assertEq(id, 3);
    assertEq(index, 2);
    assertEq(traderContract, newSwingTrader);
    assertEq(name, "Test");
    assertEq(active, false);
    vm.prank(admin);
    swingTraderManager.toggleTraderActive(3);
    assertEq(swingTraderManager.activeTraders(2),3);
    assertEq(swingTraderManager.activeTraders(3),3); // @audit:activeTraders[2] = activeTraders[3] = 3
    vm.prank(admin);
    swingTraderManager.toggleTraderActive(3);
    assertEq(swingTraderManager.activeTraders(2),3);
  }
```
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/SwingTraderManager.sol#L397-L447
## Tools Used
None
## Recommended Mitigation Steps
Change to
```diff
  function addSwingTrader(
    uint256 traderId,
    address _swingTrader,
    bool active,
    string calldata name
  ) external onlyRoleMalt(ADMIN_ROLE, "Must have admin privs") {
    SwingTraderData storage trader = swingTraders[traderId];
    require(traderId > 2 && trader.id == 0, "TraderId already used");
    require(_swingTrader != address(0), "addr(0)");

    swingTraders[traderId] = SwingTraderData({
      id: traderId,
-     index: activeTraders.length,
+     index: active ? activeTraders.length : 0,
      traderContract: _swingTrader,
      name: name,
      active: active
    });
+  if(active) activeTraders.push(traderId);

-   activeTraders.push(traderId);

    emit AddSwingTrader(traderId, name, active, _swingTrader);
  }
```