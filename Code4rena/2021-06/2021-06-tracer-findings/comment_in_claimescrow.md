## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Comment in claimEscrow](https://github.com/code-423n4/2021-06-tracer-findings/issues/26) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function claimEscrow of Liquidation.sol can be called by everyone. The claimed funds go to the trader so there are no funds at risk.
However the comment says the traders is doing this.

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Liquidation.sol#L106
/**
     * @notice Allows a trader to claim escrowed funds after the escrow period has expired
     * @param receiptId The ID number of the insurance receipt from which funds are being claimed from
     */
    function claimEscrow(uint256 receiptId) public override {


## Tools Used

## Recommended Mitigation Steps
Double check and it the code works as intended adapt the comment.
Otherwise add check that only the trader can call the function.

