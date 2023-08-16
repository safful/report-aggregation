## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary code execution can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/210) 

# Handle

Jujic


# Vulnerability details

## Impact
```
function rate() external view returns (uint256) {
        if (totalSupply() > 0) {
            return (totalLiquidity() * MAGIC_SCALE_1E6) / totalSupply();
        } else {
            return 0;
        }
    }
```

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L512-L518

## Tools Used
Remix

## Recommended Mitigation Steps
Change to:
```
function rate() external view returns (uint256) {
        if (totalSupply() != 0) {
            return (totalLiquidity() * MAGIC_SCALE_1E6) / totalSupply();
        
    }

```

