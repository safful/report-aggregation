## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Basket.sol#changeLicenseFee()` Remove redundant check can save gas](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/62) 

# Handle

WatchPug


# Vulnerability details

`Basket.sol#changeLicenseFee()` checks for `pendingLicenseFee.licenseFee  != 0`, while the assertion above already making sure that `newLicenseFee >= factory.minLicenseFee()`.

If we can make sure `factory.minLicenseFee() > 0`, then the check of `pendingLicenseFee.licenseFee != 0` will be redundant.

Removing it will make the code simpler and save some gas.

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L167-L170

```solidity
function changeLicenseFee(uint256 newLicenseFee) onlyPublisher public override {
    require(newLicenseFee >= factory.minLicenseFee() && newLicenseFee != licenseFee);
    if (pendingLicenseFee.licenseFee != 0 && pendingLicenseFee.licenseFee == newLicenseFee) {
        require(block.number >= pendingLicenseFee.block + TIMELOCK_DURATION);
```

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Factory.sol#L39-L41

```solidity
function setMinLicenseFee(uint256 newMinLicenseFee) public override onlyOwner {
    minLicenseFee = newMinLicenseFee;
}
```

### Recommendation

Consider adding `require(newMinLicenseFee > 0);` to `Factory.sol#setMinLicenseFee()`.

Remove the redundant check.

