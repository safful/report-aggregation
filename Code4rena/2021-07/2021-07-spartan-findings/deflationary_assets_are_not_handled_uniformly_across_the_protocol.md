## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Deflationary assets are not handled uniformly across the protocol](https://github.com/code-423n4/2021-07-spartan-findings/issues/101) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The DAO codebase not handle deflationary asset tokens. However, this is handled in similar _handleTransferIn functions of Router and poolFactory which indicates that protocol allows/anticipates listing of deflationary tokens which require a start balance check/subtraction before and after transfers to account for the actual amount transferred instead of taking the face-value amount from the parameter without considering any transfer fees imposed by the token contract.

Rationale for Medium severity: This is typically a low-severity finding in protocols that uniformly do not handle deflationary/inflationary/rebasing tokens because they either whitelist-away such tokens or do not anticipate handling them (by documenting and warning users) in their protocols. Spartan however has code indicative of expecting/handling deflationary tokens in Router and poolFactory but is missing similar special handling in DAO which is a case of missed handling and so is more serious because it leads to mis-accounting and potential fund loss in different parts of the protocol code.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L266

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L206-L208

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L111-L113

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add code similar to Router and poolFactory to handle deflationary tokens in DAO.

