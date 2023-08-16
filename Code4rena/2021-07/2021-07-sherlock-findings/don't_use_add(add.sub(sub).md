## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [don't use add(add.sub(sub)](https://github.com/code-423n4/2021-07-sherlock-findings/issues/23) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _updateData contains variables with the names "sub" and "add". There are also functions with the names "sub" and "add".
The resulting code is not very readable:
      usdPerBlock = usdPerBlock.sub(sub.sub(add).div(10**18));
      usdPerBlock = usdPerBlock.add(add.sub(sub).div(10**18));

Generally it is not recommended to use the same name for variables and functions.

## Proof of Concept
// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Manager.sol#L386
 function _updateData(
.. 
   uint256 sub = _oldPremium.mul(_oldUsd);
 .. 
  uint256 add = _newPremium.mul(_newUsd);

    if (sub > add) {
      usdPerBlock = usdPerBlock.sub(sub.sub(add).div(10**18));
    } else {
      usdPerBlock = usdPerBlock.add(add.sub(sub).div(10**18));
    }

## Tools Used

## Recommended Mitigation Steps
Rename the variables "add" and "sub" to different names.


