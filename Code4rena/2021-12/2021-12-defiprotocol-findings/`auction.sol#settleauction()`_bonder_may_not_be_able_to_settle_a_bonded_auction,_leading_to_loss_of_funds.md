## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Auction.sol#settleAuction()` Bonder may not be able to settle a bonded auction, leading to loss of funds](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/106) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-defiprotocol/blob/205d3766044171e325df6a8bf2e79b37856eece1/contracts/contracts/Auction.sol#L97-L102

```solidity=97
    uint256 a = factory.auctionMultiplier() * basket.ibRatio();
    uint256 b = (bondBlock - auctionStart) * BASE / factory.auctionDecrement();
    uint256 newRatio = a - b;

    (address[] memory pendingTokens, uint256[] memory pendingWeights, uint256 minIbRatio) = basket.getPendingWeights();
    require(newRatio >= minIbRatio);
```

In the current implementation, `newRatio` is calculated and compared with `minIbRatio` in `settleAuction()`.

However, if `newRatio` is less than `minIbRatio`, `settleAuction()` will always fail and there is no way for the bonder to cancel and get a refund.

### PoC

Given:

- `bondPercentDiv` = 400
- `basketToken.totalSupply` = 40,000
- `factory.auctionMultiplier` = 2
- `factory.auctionDecrement` = 10,000
- `basket.ibRatio` = 1e18
- p`endingWeights.minIbRatio` = 1.9 * 1e18

1. Alice called `bondForRebalance()` `2,000` blocks after the auction started, paid `100` basketToken for the bond;
2. Alice tries to `settleAuction()`, it will always fail because `newRatio < minIbRatio`;
- a = 2 * 1e18
- b = 0.2 * 1e18
- newRatio = 1.8 * 1e18;
3. Bob calls `bondBurn()` one day after, `100` basketToken from Alice will been burned.

### Recommendation

Move the `minIbRatio` check to `bondForRebalance()`:

```solidity=58
function bondForRebalance() public override {
        require(auctionOngoing);
        require(!hasBonded);

        bondTimestamp = block.timestamp;
        bondBlock = block.number;

        uint256 a = factory.auctionMultiplier() * basket.ibRatio();
        uint256 b = (bondBlock - auctionStart) * BASE / factory.auctionDecrement();
        uint256 newRatio = a - b;

        (address[] memory pendingTokens, uint256[] memory pendingWeights, uint256 minIbRatio) = basket.getPendingWeights();
        require(newRatio >= minIbRatio);

        IERC20 basketToken = IERC20(address(basket));
        bondAmount = basketToken.totalSupply() / factory.bondPercentDiv();
        basketToken.safeTransferFrom(msg.sender, address(this), bondAmount);
        hasBonded = true;
        auctionBonder = msg.sender;

        emit Bonded(msg.sender, bondAmount);
    }
```

