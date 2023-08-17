## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method
- valid

# [Lack of check on `mustStartAtOrAfter`](https://github.com/code-423n4/2022-07-juicebox-findings/issues/220) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBFundingCycleStore.sol#L306-L312
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBFundingCycleStore.sol#L518-L522


# Vulnerability details

## Impact

**MED** - the function of the protocol could be impacted

By setting huge `mustStartAtOrAfter`, the owner can set start time in the past. It might open up possibility to bypass the ballot waiting time depending on the ballot's implementation.

## Proof of Concept

- [proof of concept](https://gist.github.com/zzzitron/a8c6067923a87af8e001c05442258370#file-2022-07-juiceboxv2-t-sol-L77-L115)

The proof of concept is almost the same as [`TestReconfigure::testReconfigureProject`](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/system_tests/TestReconfigure.sol#L77-L114). In the original test, the owner of the project is reconfiguring funding cycle, but it is not in effect immediately because ballot is set. Only after 3 days the newly set funding cycle will be the current one.
In the above proof of concept, only one parameter of the funding cycle is modified: `mustStartAtOrAfter` is set to `type(uint56).max`. As the result, the newly set funding cycle is considered as the current one without waiting for the ballot.

The cause of this is missing check on `mustStartAtOrAfter` upon setting [here](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBFundingCycleStore.sol#L306-L312). If the given `_mustStartAtOrAfter` is huge, it will be passed eventually to the `_initFor`, `_packAndStoreIntrinsicPropertiesOf`. Then it will 'overflow' by shifting and set to the funding cycle, which [essentially can be set to any value including the past](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBFundingCycleStore.sol#L518-L522). Also, it seems like the number will be also effected because the bigger digit will carry over.

```solidity
// in JBFundingCycleStore::_packAndStoreIntrinsicPropertiesOf
// where the `_start` is derived from `_mustStartAtOrAfter`

./JBFundingCycleStore.sol-518-    // start in bits 144-199.
./JBFundingCycleStore.sol:519:    packed |= _start << 144;
./JBFundingCycleStore.sol-520-
./JBFundingCycleStore.sol-521-    // number in bits 200-255.
./JBFundingCycleStore.sol-522-    packed |= _number << 200;
```

## Tools Used

foundry

## Recommended Mitigation Steps

Add a check for the `_mustStartAtOrAfter`:
```solidity
// example check for _mustSTartAtOrAfter
// in JBFundingCycleStore::configureFor

if (_mustStartAtOrAfter > type(uint56).max) revert INVALID_START();
```




