## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Protocol doesn't handle fee on transfer tokens](https://github.com/code-423n4/2022-04-backed-findings/issues/75) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L155-L160


# Vulnerability details

## Impact
Since the borrower is able to specify any asset token, it is possible that loans will be created with tokens that support fee on transfer. If a fee on transfer asset token is chosen, the protocol will contain a point of failure on the original `lend()` call.

It is my belief that this is a medium severity vulnerability due to its ability to impact core protocol functionality.

## Proof of Concept

For the first lender to call `lend()`, if the transfer fee % of the asset token is larger than the origination fee %, the second transfer will fail in the following code:

```
            ERC20(loanAssetContractAddress).safeTransferFrom(msg.sender, address(this), amount);
            uint256 facilitatorTake = amount * originationFeeRate / SCALAR;
            ERC20(loanAssetContractAddress).safeTransfer(
                IERC721(borrowTicketContract).ownerOf(loanId),
                amount - facilitatorTake
            );
```

Example:
- `originationFee = 2%` Max fee is 5% per comments
- `feeOnTransfer = 3%`
- `amount = 100 tokens`

- Lender transfers `amount`
- `NFTLoanFacilitator` receives `97`.
- `facilitatorTake = 2`
- `NFTLoanFacilitator` attempts to send `100 - 2` to borrower, but only has `97`.
- Execution reverts.

### Other considerations:
If the originationFee is less than or equal to the transferFee, the transfers will succeed but will be received at a loss for the borrower and lender. Specifically for the lender, it might be unwanted functionality for a lender to lend 100 and receive 97 following a successful repayment (excluding interest for this example).

## Tools Used
Manual review.

## Recommended Mitigation Steps
Since the `originationFee` is calculated based on the `amount` sent by the lender, this calculation will always underflow given the example above. Instead, a potential solution would be to calculate the `originationFee` based on the requested loan amount, allowing the lender to send a greater value so that `feeOnTransfer <= originationFee`.

Oppositely, the protocol can instead calculate the amount received from the initial transfer and use this amount to calculate the `originationFee`. The issue with this option is that the borrower will receive less than the desired loan amount.

