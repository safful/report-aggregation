## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Timelock

# [double negative in comment](https://github.com/code-423n4/2021-08-yield-findings/issues/1) 

# Handle

gpersoon


# Vulnerability details

## Impact
TimeLock.sol contains a comment with a double negative, which is confusing to read:
Transaction not unknown.

## Proof of Concept
//https://github.com/code-423n4/2021-08-yield/blob/main/contracts/utils/TimeLock.sol#L55
 function schedule(address[] calldata targets, bytes[] calldata data, uint256 eta)   external override auth returns (bytes32 txHash)  {
..
        require(transactions[txHash] == 0, "Transaction not unknown.");

## Tools Used

## Recommended Mitigation Steps
Replace:
Transaction not unknown.
with:
Transaction already scheduled.

