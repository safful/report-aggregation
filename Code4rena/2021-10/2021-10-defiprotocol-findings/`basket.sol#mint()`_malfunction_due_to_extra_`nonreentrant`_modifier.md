## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Basket.sol#mint()` Malfunction due to extra `nonReentrant` modifier](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/59) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L83-L88

```solidity
function mint(uint256 amount) public nonReentrant override {
    mintTo(amount, msg.sender);
}

function mintTo(uint256 amount, address to) public nonReentrant override {
    require(auction.auctionOngoing() == false);
```

The `mint()` method is malfunction because of the extra `nonReentrant` modifier, as `mintTo` already has a `nonReentrant` modifier.

### Recommendation

Change to:

```solidity
function mint(uint256 amount) public override {
    mintTo(amount, msg.sender);
}
```

