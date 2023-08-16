## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [wrapCall with weird ERC20 contracts](https://github.com/code-423n4/2021-07-connext-findings/issues/4) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function wrapCall is not completely safe for all possible ERC20 contracts.

If the returnData.length is larger than 1 the "abi.decode(returnData, (bool));" will fail.
Which means the interactions with that ERC20 contract will fail.
Although this is unlikely, it is easy to protect against it.

## Proof of Concept
// https://github.com/code-423n4/2021-07-connext/blob/main/contracts/lib/LibERC20.sol#L21
    function wrapCall(address assetId, bytes memory callData) internal returns (bool) {
        ...
        (bool success, bytes memory returnData) = assetId.call(callData);
        LibUtils.revertIfCallFailed(success, returnData);
        return returnData.length == 0 || abi.decode(returnData, (bool));
    }

## Tools Used

## Recommended Mitigation Steps
Change return returnData.length == 0 || abi.decode(returnData, (bool));
to:
return (returnData.length == 0) || (returnData.length == 1 && abi.decode(returnData, (bool)));

