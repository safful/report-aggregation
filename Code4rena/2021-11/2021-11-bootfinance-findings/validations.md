## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Validations](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/301) 

# Handle

pauliax


# Vulnerability details

## Impact

function burnEtherForMember should validate that the address of the member is not empty (0x0) to prevent accidental burns. 

When adding an investor distribution (function addInvestor) should validate that the total amount is not above the investors_supply. but then you also need to store the total amount that is already assigned to investors. 

function modifyInvestor should validate that _investor != _new, otherwise it will delete the investor unless this is an expected feature. 

function claimExact should validate that _value > 0 to prevent useless claims.

