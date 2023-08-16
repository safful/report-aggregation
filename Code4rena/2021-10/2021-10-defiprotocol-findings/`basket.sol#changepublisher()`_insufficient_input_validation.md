## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`Basket.sol#changePublisher()` Insufficient input validation](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/61) 

# Handle

WatchPug


# Vulnerability details

As per the test, changePublisher to the current publisher should not be allowed:

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/test/Basket.test.js#L122-L122

```javascript
let publisher = await basket.publisher();
expect(publisher).to.equal(addr2.address);

await expect(basket.connect(addr2).changePublisher(addr2.address)).to.be.reverted;
```

However, there is no such check to make sure that.

https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Basket.sol#L147-L148

```solidity
function changePublisher(address newPublisher) onlyPublisher public override {
    require(newPublisher != address(0));
```

### Recommendation

Change to:

```solidity
function changePublisher(address newPublisher) onlyPublisher public override {
    require(newPublisher != address(0) && newPublisher != publisher);
```

