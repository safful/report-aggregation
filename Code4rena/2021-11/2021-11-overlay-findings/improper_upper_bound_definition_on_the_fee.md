## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Improper Upper Bound Definition on the Fee](https://github.com/code-423n4/2021-11-overlay-findings/issues/77) 

# Handle

defsec


# Vulnerability details

## Impact

In the adjustGlobalParams function on line 1603of "https://github.com/code-423n4/2021-11-overlay/blob/main/contracts/mothership/OverlayV1Mothership.sol#L1630", adjustGlobalParams function does not have any upper or lower bounds. Values that are too large will lead to reversions in several critical functions.

## Proof of Concept

- The setFee function that begins on line 163 of adjustGlobalParams sets the liquidity and transaction fee rates for the market in which the function is called. In this context, the transaction fee is the percentage of a transaction that is taken by the protocol and moved to a designated reserve account. As the name suggests, transaction fees factor in to many of the essential transaction types performed within the system.
- Navigate to "https://github.com/code-423n4/2021-11-overlay/blob/main/contracts/mothership/OverlayV1Mothership.sol#L163" contract and go to line #163.
- On the function there is no upper and lower bound defined. Therefore, users can pay higher fees.

## Tools Used

None

## Recommended Mitigation Steps

Consider to define upper and lower bounds on the adjustGlobalParams function.

