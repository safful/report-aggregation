## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [sqrt can be made unchecked to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/26) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

gas costs

## Proof of Concept

In the sqrt function it is known that the while loop will not overflow so it can be safely left unchecked to save gas.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PremiumModels/BondingPremium.sol#L238-L245

```
    function sqrt(uint256 x) internal pure returns (uint256 y) {
        uint256 z = (x + 1) / 2;
        unchecked {
            y = x;
            while (z < y) {
                y = z;
                z = (x / z + z) / 2;
            }
        }
    }
```
## Recommended Mitigation Steps

wrap entire function body in a unchecked block as above

