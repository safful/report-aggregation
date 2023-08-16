## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [getEmissionShare Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/28) 

# Handle

ye0lde


# Vulnerability details

## Impact

getEmissionShare Can Be Rewritten To Be More Efficient (PublicSale.sol)

The "else" and returning 0, can be eliminated.  The existing but unused named return variable "value" can be used instead of a return statement.
These changes reduce gas and improve code clarity.

## Proof of Concept

The getEmissionShare function is here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L231-L245

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
I recommend rewriting it as follows:
<code>
    function getEmissionShare(uint era, uint day, address member) public view returns (uint value) {  
        uint memberUnits = mapEraDay_MemberUnits[era][day][member];                         // Get Member Units
        
        if (memberUnits != 0) {
            uint totalUnits = mapEraDay_UnitsRemaining[era][day];                           // Get Total Units
            uint emissionRemaining = mapEraDay_EmissionRemaining[era][day];     // Get emission remaining for Day
            uint balance = mainToken.balanceOf(address(this));
            if (emissionRemaining > balance) {
                emissionRemaining = balance;                                                // In case less than required emission
            }
            value = (emissionRemaining * memberUnits) / totalUnits;         // Calculate share
        }
    }   
</code>

