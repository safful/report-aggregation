## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`DutchAuctionLiquidator.sol#triggerLiquidation()` Adding precondition check can save gas](https://github.com/code-423n4/2021-10-mochi-findings/issues/104) 

# Handle

WatchPug


# Vulnerability details

When liquidators race to liquidate a position, all other besides the first liquidator will be handling an empty (liquidated) position.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/liquidator/DutchAuctionLiquidator.sol#L69-L81

```solidity=69
function triggerLiquidation(address _asset, uint256 _nftId)
    external
    override
{
    IMochiVault vault = engine.vaultFactory().getVault(_asset);
    Auction storage auction = auctions[auctionId(_asset, _nftId)];
    require(auction.startedAt == 0 || auction.boughtAt != 0, "on going");
    uint256 debt = vault.currentDebt(_nftId);

    (, uint256 collateral, , , ) = vault.details(_nftId);

    vault.liquidate(_nftId, collateral, debt);
    ...
```

In the current implementation, even if the position is liquidated, at L77 and L79, it still tries to get the details and call `vault.liquidate()`, until it reverts at L285-L288 on `MochiVault.sol#liquidate()`. That's going to cost a decent amount of gas due to these unnecessary external calls and code executions.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/vault/MochiVault.sol#L277-L288

```solidity=277{285-288}
function liquidate(
    uint256 _id,
    uint256 _collateral,
    uint256 _usdm
) external override updateDebt(_id) {
    require(msg.sender == address(engine.liquidator()), "!liquidator");
    require(engine.nft().asset(_id) == address(asset), "!asset");
    float memory price = engine.cssr().getPrice(address(asset));
    require(
        _liquidatable(details[_id].collateral, price, currentDebt(_id)),
        "healthy"
    );
    ...
```

Therefore, adding a precondition check can save gas.



### Recommendation

Change to:

```solidity=69{77}
function triggerLiquidation(address _asset, uint256 _nftId)
    external
    override
{
    IMochiVault vault = engine.vaultFactory().getVault(_asset);
    Auction storage auction = auctions[auctionId(_asset, _nftId)];
    require(auction.startedAt == 0 || auction.boughtAt != 0, "on going");
    uint256 debt = vault.currentDebt(_nftId);
    require(debt > 0, "!debt");
    (, uint256 collateral, , , ) = vault.details(_nftId);

    vault.liquidate(_nftId, collateral, debt);
    ...
```

