## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Math's operations order in Swivel's functions](https://github.com/code-423n4/2021-09-swivel-findings/issues/162) 

# Handle

0xsanson


# Vulnerability details

## Impact
There are multiple instances of divisions performed before multiplications, whereas the opposite is generally suggested. To mitigate the precision loss, a factor like `1e18` is multiplied and then divided, but this solution is arbitrary and can be avoided.

For example:
```js
uint256 principalFilled = (((a * 1e18) / o.premium) * o.principal) / 1e18;
```

can be rewritten like:
```js
uint256 principalFilled = a * o.principal / o.premium;
```

## Proof of Concept
Run `grep '1e18' Swivel.sol` for a complete list.

## Tools Used
grep, editor

## Recommended Mitigation Steps
Suggested checking all instances and trying to simplify the math.

