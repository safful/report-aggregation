## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization: Struct layout](https://github.com/code-423n4/2021-10-mochi-findings/issues/30) 

# Handle

gzeon


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/interfaces/IMochiVault.sol
L6-L12: the struct can be reordered into
struct Detail {
    Status status;
    address referrer;
    uint256 collateral;
    uint256 debt;
    uint256 debtIndex;
}

such that status and referrer are put into the same slot, should save ~2000 gas per borrow

## Tools Used
None

## Recommended Mitigation Steps
Reorder the struct as suggested, and modify impacted code at 
IMochiVault.sol L28-L34
DutchAuctionLiquidator.sol L77

## Extra
Realistically, the range of debtIndex (start at 1e18 and increase by fee per year) probably fit in a uint88(11bytes) so you can pack (status(1byte), referrer(20bytes), debtIndex(11bytes)) all in 32 bytes, saving another storage slot. 

