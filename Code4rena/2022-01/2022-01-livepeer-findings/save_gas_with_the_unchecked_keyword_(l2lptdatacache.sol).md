## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Save Gas With The Unchecked Keyword (L2LPTDataCache.sol)](https://github.com/code-423n4/2022-01-livepeer-findings/issues/173) 

# Handle

ye0lde


# Vulnerability details

## Impact
Save Gas With The Unchecked Keyword (L2LPTDataCache.sol)

Redundant arithmetic underflow/overflow checks can be avoided when an underflow/overflow cannot happen.

## Proof of Concept
The "unchecked" keyword can be applied here since there is an `if` statement before to ensure the arithmetic operations would not cause an integer underflow or overflow.:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2LPTDataCache.sol#L57-L69

Change the code to:

```
    function decreaseL2SupplyFromL1(uint256 _amount) external onlyL2LPTGateway {
        // If there is a mass withdrawal from L2, _amount could exceed l2SupplyFromL1.
        // In this case, we just set l2SupplyFromL1 = 0 because there will be no more supply on L2
        // that is from L1 and the excess (_amount - l2SupplyFromL1) is inflationary LPT that was
        // never from L1 in the first place.
        unchecked {
            if (_amount > l2SupplyFromL1) {
                l2SupplyFromL1 = 0;
            } else {
                l2SupplyFromL1 -= _amount;  // @audit unchecked
            }
        }

        // No event because the L2LPTGateway events are sufficient
    }
 
```

A similar change can be made here:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2LPTDataCache.sol#L91-L94

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Add the "unchecked" keyword as shown above.

