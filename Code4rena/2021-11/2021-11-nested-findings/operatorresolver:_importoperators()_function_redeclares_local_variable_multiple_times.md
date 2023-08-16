## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [OperatorResolver: importOperators() function redeclares local variable multiple times](https://github.com/code-423n4/2021-11-nested-findings/issues/119) 

# Handle

GreyArt


# Vulnerability details

## Impact

The `importOperators()` declares the `name` and `destination` variables multiple times. It'll be cheaper to declare them once outside the loop and reuse the variables inside. 

## Recommended Mitigation Steps

```jsx
bytes32 name;
address destination;

for (uint256 i = 0; i < names.length; i++) {
	name = names[i];
	destination = destinations[i];
  operators[name] = destination;
  emit OperatorImported(name, destination);
}
```

