## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- old-submission-method

# [TRSRY: front-runnable `setApprovalFor`](https://github.com/code-423n4/2022-08-olympus-findings/issues/410) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/modules/TRSRY.sol#L64-L72
https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/TreasuryCustodian.sol#L42-L48


# Vulnerability details

## Impact

An attacker may be able to withdraw more than intended

## Proof of Concept

Let's say the alice had approval of 100. Now the treasury custodian reduced the approval to 50. Alice could frontrun the `setApprovalFor` of 50, and withdraw 100 as it was before. Then withdraw 50 with the newly set approval. So the alice could withdraw 150.

```solidity
// modules/TRSRY.sol

 63     /// @notice Sets approval for specific withdrawer addresses
 64     function setApprovalFor(
 65         address withdrawer_,
 66         ERC20 token_,
 67         uint256 amount_
 68     ) external permissioned {
 69         withdrawApproval[withdrawer_][token_] = amount_;
 70
 71         emit ApprovedForWithdrawal(withdrawer_, token_, amount_);
 72     }
```

The `TreasuryCustodian` simply calls the `setApprovalFor` to grant Approval.
```solidity
 41
 42     function grantApproval(
 43         address for_,
 44         ERC20 token_,
 45         uint256 amount_
 46     ) external onlyRole("custodian") {
 47         TRSRY.setApprovalFor(for_, token_, amount_);
 48     }
```


## Tools Used

none

## Recommended Mitigation Steps

Instead of setting the given amount, one can reduce from the current approval. By doing so, it checks whether the previous approval is spend.

<!-- zzzitron M06 -->



