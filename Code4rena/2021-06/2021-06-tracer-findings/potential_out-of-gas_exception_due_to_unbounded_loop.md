## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Potential Out-of-Gas exception due to unbounded loop](https://github.com/code-423n4/2021-06-tracer-findings/issues/69) 

# Handle

0xRajeev


# Vulnerability details

## Impact
Trading function executeTrade() batch executes maker/taker orders against a market. The trader/interface provides arrays of makers/takers which is unbounded. As a result, if the number of orders is too many, there is a risk of this transaction exceeding the block gas limit (which is 15 million currently).

Impact: executeTrade() is called with too many orders in the batch. Tx exceeds block gas limit and reverts. None of the orders are executed.

## Proof of Concept

See similar Medium-severity finding from ConsenSys's Audit of Growth DeFi: https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#potential-resource-exhaustion-by-external-calls-performed-within-an-unbounded-loop

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Trader.sol#L67

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Trader.sol#L78

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Limit the number or orders executed based on gasleft() after every iteration or estimate the gas cost and enforce an upper bound on the number of orders allowed in maker/taker arrays.

