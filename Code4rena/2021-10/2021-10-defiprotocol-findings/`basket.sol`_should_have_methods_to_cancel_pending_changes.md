## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`Basket.sol` should have methods to cancel pending changes](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/60) 

# Handle

WatchPug


# Vulnerability details

While changing publisher and licenseFee is timelocked, there are no methods to cancel pending changes.

As a result, wrong changes may not be able to get canceled

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L147-L163

```solidity
function changePublisher(address newPublisher) onlyPublisher public override {
    require(newPublisher != address(0));

    if (pendingPublisher.publisher != address(0) && pendingPublisher.publisher == newPublisher) {
        require(block.number >= pendingPublisher.block + TIMELOCK_DURATION);
        publisher = newPublisher;

        pendingPublisher.publisher = address(0);

        emit ChangedPublisher(publisher);
    } else {
        pendingPublisher.publisher = newPublisher;
        pendingPublisher.block = block.number;

        emit NewPublisherSubmitted(newPublisher);
    }
}
```

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L167-L182

```solidity
function changeLicenseFee(uint256 newLicenseFee) onlyPublisher public override {
    require(newLicenseFee >= factory.minLicenseFee() && newLicenseFee != licenseFee);
    if (pendingLicenseFee.licenseFee != 0 && pendingLicenseFee.licenseFee == newLicenseFee) {
        require(block.number >= pendingLicenseFee.block + TIMELOCK_DURATION);
        licenseFee = newLicenseFee;

        pendingLicenseFee.licenseFee = 0;

        emit ChangedLicenseFee(licenseFee);
    } else {
        pendingLicenseFee.licenseFee = newLicenseFee;
        pendingLicenseFee.block = block.number;

        emit NewLicenseFeeSubmitted(newLicenseFee);
    }
}
```

### Recommendation

Consider adding methods to cancel pending changes.

