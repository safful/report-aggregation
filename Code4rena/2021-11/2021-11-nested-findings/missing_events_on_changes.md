## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing events on changes](https://github.com/code-423n4/2021-11-nested-findings/issues/84) 

# Handle

palina


# Vulnerability details

## Impact
Function performing important changes to contract state should emit events to facilitate monitoring of the protocol operation (e.g., NestedRecords::setReserve(), deleteAsset(), removeNFT()).

## Proof of Concept
setReserve(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedRecords.sol#L201
deleteAsset(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedRecords.sol#L207
removeNFT(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedRecords.sol#L221

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Consider emitting events on the discussed changes. E.g., 
event ReserveUpdated(address newReserve);
...
function setReserve(...) {
    emit ReserveUpdated(_nextReserve);
}

