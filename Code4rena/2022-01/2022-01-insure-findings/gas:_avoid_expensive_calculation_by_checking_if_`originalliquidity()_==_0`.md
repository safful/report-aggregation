## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Avoid expensive calculation by checking if `originalLiquidity() == 0`](https://github.com/code-423n4/2022-01-insure-findings/issues/361) 

# Handle

Dravee


# Vulnerability details

## Impact
Checking if the value is 0 before returning 0 is less expensive than returning a calculation that's equal to 0

## Proof of Concept
In `PoolTemplate.sol:rate()`, the code is as follows:
```
File: PoolTemplate.sol
744:     function rate() external view returns (uint256) {
745:         if (totalSupply() > 0) {
746:             return (originalLiquidity() * MAGIC_SCALE_1E6) / totalSupply();
747:         } else {
748:             return 0;
749:         }
750:     }

```
It can be optimized as such:
```
744:     function rate() external view returns (uint256) {
745:         uint256 originalLiquidity = originalLiquidity();
746:         if (originalLiquidity != 0 && totalSupply() > 0) {
747:             return (originalLiquidity * MAGIC_SCALE_1E6) / totalSupply();
748:         } else {
749:             return 0;
750:         }
751:     }

```

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache the loaded storage value in a memory variable and make the 0 checks to avoid unnecessary calculations if `originalLiquidity() == 0`

