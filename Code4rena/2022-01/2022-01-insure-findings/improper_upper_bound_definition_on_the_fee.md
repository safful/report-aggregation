## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor acknowledged
- sponsor confirmed

# [Improper Upper Bound Definition on the Fee](https://github.com/code-423n4/2022-01-insure-findings/issues/229) 

# Handle

defsec


# Vulnerability details

The setFeeRate function does not have any upper or lower bounds. Values that are too large will lead to reversions in several critical functions.

## Proof of Concept

- The setFeeRate function sets the transaction fee rates for the market in which the function is called. In this context, the transaction fee is the percentage of a transaction that is taken by the protocol and moved to a designated reserve account. As the name suggests, transaction fees factor in to many of the essential transaction types performed within the system.
- Navigate to "https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Parameters.sol#L177".
- On the function there is no upper and lower bound defined. Therefore, users can pay higher fees.

## Tools Used

None

## Recommended Mitigation Steps

Consider to define upper and lower bounds on the fee array.

