## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [`SwingTraderManager.swingTraders()` shoudn't contain duplicate `traderContract`s.](https://github.com/code-423n4/2023-02-malt-findings/issues/34) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/StabilityPod/SwingTraderManager.sol#L36


# Vulnerability details

## Impact
If `SwingTraderManager.swingTraders()` contains duplicate `traderContract`s, several functions like `buyMalt()` and `sellMalt()` wouldn't work as expected as they work according to traders' balances.

## Proof of Concept
During the swing trader addition, there is no validation that each trader should have a unique `traderContract`.

```solidity
  function addSwingTrader(
    uint256 traderId,
    address _swingTrader, //@audit should be unique
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

So the same `traderContract` might have 2 or more `traderId`s.

When we check `buyMalt()` as an example, it distributes the ratio according to the trader balance and it wouldn't work properly if one trader contract is counted twice and receives more shares that it can't manage.

Similarly, other functions wouldn't work as expected and return the wrong result.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Recommend adding a new mapping like `activeTraderContracts` to check if the contract is added already or not.

Then we can check the trader contract is added only once.