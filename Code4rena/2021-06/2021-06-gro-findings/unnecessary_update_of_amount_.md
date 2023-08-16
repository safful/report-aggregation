## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary update of amount ](https://github.com/code-423n4/2021-06-gro-findings/issues/18) 

# Handle

gpersoon


# Vulnerability details

## Impact
In several functions of BaseVaultAdaptor a value is stored in the variable amount at the end of the function.
However this variable is never used afterwards so the storage is unnecessary and just uses gas.

## Proof of Concept
// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/vaults/BaseVaultAdaptor.sol#L165
    function withdraw(uint256 amount) external override {
     ..
        if (!_withdrawFromAdapter(amount, msg.sender)) {
            amount = _withdraw(calculateShare(amount), msg.sender);

    function withdraw(uint256 amount, address recipient) external override {
    ...
        if (!_withdrawFromAdapter(amount, recipient)) {
            amount = _withdraw(calculateShare(amount), recipient);

    function withdrawToAdapter(uint256 amount) external onlyOwner {
        amount = _withdraw(calculateShare(amount), address(this));
    }

    function withdrawByStrategyOrder(
..
        if (!_withdrawFromAdapter(amount, recipient)) {
            amount = _withdrawByStrategyOrder(calculateShare(amount), recipient, reversed);

    function withdrawByStrategyIndex(
   ...
        if (!_withdrawFromAdapter(amount, recipient)) {
            amount = _withdrawByStrategyIndex(calculateShare(amount), recipient, strategyIndex);


## Tools Used

## Recommended Mitigation Steps
Replace
  amount = _withdraw***(...);
with
   _withdraw***(...);

