## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Dao.sol: Insufficient validation for proposal creation](https://github.com/code-423n4/2021-07-spartan-findings/issues/43) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In general, creating invalid proposals is easy due to the lack of validation in the `new*Proposal()` functions.

- The `typeStr` is not validated at all. For example, one can call `newActionProposal()` with `typeStr = ROUTER` or `typeStr = BAD_STRING`, both of which will pass. The first will cause `finaliseProposal()` to fail because the proposed address is null, preventing `completeProposal()` from executing. The second does nothing because it does not equate to any of the check `typeStr`, and so `completeProposal()` isn't executed at all.
- Not checking the proposed values are null. The checks only happen in `finaliseProposal()` when the relevant sub-functions are called, like the `move*()` functions.

All of these scenarios lead to a mandatory 15 day wait since proposal creation in order to be cancelled, which prevents the creation of new proposals (in order words, denial of service of the DAO).

### Recommended Mitigation Steps

1. Since the number of proposal types is finite, it is best to restrict and validate the `typeStr` submitted. Specifically,
    - `newActionProposal()` should only allow `FLIP_EMISSIONS` and `GET_SPARTA` proposal types
    - `newAddressProposal()` should only allow `DAO`, `ROUTER`, `UTILS`, `RESERVE`, `LIST_BOND`, `DELIST_BOND`, `ADD_CURATED_POOL` and  `REMOVE_CURATED_POOL` proposal types
    - `newParamProposal()` should only allow `COOL_OFF` and `ERAS_TO_EARN` proposal types
2. Perhaps have a "catch-all-else" proposal that will only call `_completeProposal()` in `finaliseProposal()`

```jsx
function finaliseProposal() external {
	...
	} else if (isEqual(_type, 'ADD_CURATED_POOL')){
		_addCuratedPool(currentProposal);
  } else if (isEqual(_type, 'REMOVE_CURATED_POOL')){
    _removeCuratedPool(currentProposal);
  } else {
		completeProposal(_proposalID);
	}
}
```

3. Do null validation checks in `newAddressProposal()` and `newParamProposal()`

```jsx
function newAddressProposal(address proposedAddress, string memory typeStr) external returns(uint) {
    require(proposedAddress != address(0), "!address");
		// TODO: validate typeStr
		...
}

function newParamProposal(uint32 param, string memory typeStr) external returns(uint) {
    require(param != 0, "!param");
		// TODO: validate typeStr
		...
}
```

