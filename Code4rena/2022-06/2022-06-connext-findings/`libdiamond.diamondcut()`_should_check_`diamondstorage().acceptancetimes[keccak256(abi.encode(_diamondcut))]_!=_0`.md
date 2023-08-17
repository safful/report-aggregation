## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`LibDiamond.diamondCut()` should check `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] != 0`](https://github.com/code-423n4/2022-06-connext-findings/issues/215) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L100-L103
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L71-L79
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L83-L90


# Vulnerability details

## Impact

Normally, `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))]` will be set in `LibDiamond.proposeDiamondCut()`. Then in `LibDiamond.diamondCut()`, it checks that `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] < block.timestamp`.

However, `LibDiamond.rescindDiamondCut()` will set `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))]` to 0. Which can easily pass the check in `diamondCut()`. But `rescindDiamondCut` should rescind `_diamondCut`. In conclusion, using `rescindDiamondCut()` can easily bypass the delay time.

Moreover, if `proposeDiamondCut()` has never been called, the check for delay time is always passed.

## Proof of Concept

`diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))]` will be set in `LibDiamond.proposeDiamondCut()`
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L71-L79
```
  function proposeDiamondCut(
    IDiamondCut.FacetCut[] memory _diamondCut,
    address _init,
    bytes memory _calldata
  ) internal {
    uint256 acceptance = block.timestamp + _delay;
    diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] = acceptance;
    emit DiamondCutProposed(_diamondCut, _init, _calldata, acceptance);
  }
```

Then in `LibDiamond.diamondCut()`, it checks that `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] < block.timestamp`
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L100-L103
```
  function diamondCut(
    IDiamondCut.FacetCut[] memory _diamondCut,
    address _init,
    bytes memory _calldata
  ) internal {
    require(
      diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] < block.timestamp,
      "LibDiamond: delay not elapsed"
    );
    …
  }
```

However, `LibDiamond.rescindDiamondCut()` will set `diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))]` to 0. Which can easily pass the check in `diamondCut()`
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L83-L90
```
  function rescindDiamondCut(
    IDiamondCut.FacetCut[] memory _diamondCut,
    address _init,
    bytes memory _calldata
  ) internal {
    diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] = 0;
    emit DiamondCutRescinded(_diamondCut, _init, _calldata);
  }
```

```
diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] = 0 < block.timestamp
```


## Tools Used

None

## Recommended Mitigation Steps

Add another check in `diamondCut`

```
  function diamondCut(
    IDiamondCut.FacetCut[] memory _diamondCut,
    address _init,
    bytes memory _calldata
  ) internal {
    require(
      diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] < block.timestamp && diamondStorage().acceptanceTimes[keccak256(abi.encode(_diamondCut))] != 0,
      "LibDiamond: delay not elapsed"
    );
    …
  }
```


