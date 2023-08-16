## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Basket.sol#changePublisher()` Remove redundant assertion can save gas](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/64) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L147-L152

```solidity
function changePublisher(address newPublisher) onlyPublisher public override {
        require(newPublisher != address(0));

        if (pendingPublisher.publisher != address(0) && pendingPublisher.publisher == newPublisher) {
            require(block.number >= pendingPublisher.block + TIMELOCK_DURATION);
            publisher = newPublisher;
```

`pendingPublisher.publisher` will never be `address(0)` if `newPublisher != address(0)` and `pendingPublisher.publisher == newPublisher`.

Removing `pendingPublisher.publisher != address(0)` can make the code simpler and save some gas.

### Recommendation

Remove the redundant assertion.

