## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary `SLOAD`s and `MLOAD`s in for-each loops](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/36) 

# Handle

pants


# Vulnerability details

There are many for loops that follows this for-each pattern:
```
for (uint256 i = 0; i < array.length; i++) {
	// do something with `array[i]`
}
```

In such for loops, the `array.length` is read on every iteration, instead of caching it once in a local variable and read it from there.

## Impact
Storage reads are much more expensive than reading local variables. Memory reads are a bit more expensive than reading local variables.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Read these values from storage / memory once, cache them in local variables and then read them again from the local variables. For example:
```
uint256 length = array.length;
for (uint256 i = 0; i < length; i++) {
	// do something with `array[i]`
}
```

