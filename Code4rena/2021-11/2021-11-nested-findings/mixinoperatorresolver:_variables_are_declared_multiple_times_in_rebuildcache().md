## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [MixinOperatorResolver: variables are declared multiple times in rebuildCache()](https://github.com/code-423n4/2021-11-nested-findings/issues/122) 

# Handle

GreyArt


# Vulnerability details

## Impact

The `name` and `destination` local variables in the `rebuildCache` function are declared multiple times within the loop. It'll be cheaper to declare them once outside the loop and reuse the variables inside. 

## Recommended Mitigation Steps

```jsx
bytes32 name;
address destination;
// The resolver must call this function whenever it updates its state
for (uint256 i = 0; i < requiredAddresses.length; i++) {
	name = requiredAddresses[i];
	// Note: can only be invoked once the resolver has all the targets needed added
	destination = resolver.getAddress(name);
  ...
}
```

