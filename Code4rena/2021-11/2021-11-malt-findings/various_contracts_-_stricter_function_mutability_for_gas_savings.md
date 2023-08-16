## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Various contracts - stricter function mutability for gas savings](https://github.com/code-423n4/2021-11-malt-findings/issues/330) 

# Handle

ScopeLift


# Vulnerability details

## Impact

There are four functions that can have stricter function mutability declarations. Using a stricter declaration can help the compiler save gas when it knows whether reads and writes will occur in a function

## Proof of Concept

N/A

## Tools Used

solc

## Recommended Mitigation Steps

Implement the changes suggested by the Solidity compiler. For examples like `AbstractTransferVerification`, the solidity compiler is wrong because it doesn't know you plan to override this function declaration. Instead, `AbstractTransferVerification` could be an interface without a function definition

```
contracts/AbstractTransferVerification.sol:9:3: Warning: Function state mutability can be restricted to pure
  function verifyTransfer(address from, address to, uint256 amount) public view virtual returns (bool, string memory) {
  ^ (Relevant source part starts here and spans across multiple lines).

contracts/AuctionEscapeHatch.sol:168:3: Warning: Function state mutability can be restricted to view
  function _calculateMaltRequiredForExit(uint256 _auctionId, uint256 amount) internal returns(uint256) {
  ^ (Relevant source part starts here and spans across multiple lines).

contracts/AuctionParticipant.sol:127:3: Warning: Function state mutability can be restricted to pure
  function _handleRewardDistribution(uint256 rewarded) virtual internal {
  ^ (Relevant source part starts here and spans across multiple lines).

contracts/MaltDataLab.sol:202:3: Warning: Function state mutability can be restricted to pure
  function _normalizedPrice(
  ^ (Relevant source part starts here and spans across multiple lines).
```

