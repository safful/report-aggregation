## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [double storage call in function decreaseL2SupplyFromL1](https://github.com/code-423n4/2022-01-livepeer-findings/issues/51) 

# Handle

Tomio


# Vulnerability details

## Impact
save in memory can save more gas instead of double storage call

## Proof of Concept
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2LPTDataCache.sol#L57

## Tools Used
Remix

## Recommended Mitigation Steps
add `l2SupplyFromL1` to memory
example:
```
        uint256 savel2SupplyFromL1 = l2SupplyFromL1;
        if (_amount > savel2SupplyFromL1) {
            savel2SupplyFromL1 = 0;
        } else {
            savel2SupplyFromL1 -= _amount;
        }
```

