## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Resolved

# [RCTreasury: new hasRole() function with string role](https://github.com/code-423n4/2021-08-realitycards-findings/issues/45) 

# Handle

hickuphh3


# Vulnerability details

### Impact

It might be operationally easier (eg. reading permissions via etherscan, saves a few seconds having to search for the bytes32 constant and copy its value) to take in the role of type `string` instead of `bytes32`. It also has an added benefit of overloading the `hasRole()` function, where overriding was desired.

### Recommended Mitigation Steps

Perhaps add it as an additional function to avoid having to change all `treasury.checkPermissions()` function calls (since the role input has to be modified too).

```jsx
// TODO: add to interface
function hasRole(string memory role, address account)
	external
	view
	override
	returns (bool)
{
	bytes32 _role = keccak256(abi.encodePacked(role));
	return AccessControl.hasRole(_role, account);
}
```

