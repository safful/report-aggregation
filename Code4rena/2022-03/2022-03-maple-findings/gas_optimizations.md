## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- sponsor disputed

# [Gas Optimizations](https://github.com/code-423n4/2022-03-maple-findings/issues/35) 

1. Use `!= 0` instead of `> 0` to Save Gas

Proof of Concept:
MapleLoan.sol#L201
Migrator.sol#L25

Recommended Mitigation Steps:
Change `>` to `!=` for small gas savings.

========================================================================

2. More efficient gas usage by removing && operator

Proof of Concept:
MapleLoanInternals.sol#L296
MapleLoanInternals.sol#L361-L364
MapleLoan.sol#L233

Recommended Mitigation Steps:
Change to:
```
require((_nextPaymentDueDate == uint256(0)), "MLI:FL:LOAN_ACTIVE");
require((paymentsRemaining != uint256(0)), "MLI:FL:LOAN_ACTIVE");
```
========================================================================

3. The default of uint is already 0

Proof of Concept:
MapleLoanInternals.sol#L369-L371

Recommended Mitigation Steps:
considered remove 0 value can save gas

========================================================================

4. considered using bool in `modifier lock()` can save gas

Proof of Concept:
RevenueDistributionToken.sol#L38

Recommended Mitigation Steps:
Example:
```
pragma solidity =0.8.7;

contract test {

bool internal _locked = true;
uint256 internal _lock = 1;

modifier noReenter() {
      require(_locked, "LOCKED");
      _locked = true;
      _;
      _locked = false;
  }
modifier noReentir(){
      require(_lock == 1, "LOCKED");
      _lock = 2;
      _;
      _lock = 1;
}

  function abc() public noReenter returns(uint){
      return 12121;
      // 22066
  }
  function def() public noReentir returns(uint){
      return 1234;
      // 23752
  }
}
```

========================================================================
