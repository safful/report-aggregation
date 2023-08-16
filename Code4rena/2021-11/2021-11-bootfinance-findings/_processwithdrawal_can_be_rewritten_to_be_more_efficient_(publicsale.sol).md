## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [_processWithdrawal Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/27) 

# Handle

ye0lde


# Vulnerability details

## Impact

_processWithdrawal Can Be Rewritten To Be More Efficient (PublicSale.sol)

The "else", the setting of "value to 0", and the return statement can be eliminated to reduce gas and improve code clarity.

## Proof of Concept

The _processWithdrawal function is here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L212-L229

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
I recommend rewriting it as follows:
<code>
    function _processWithdrawal (uint _era, uint _day, address _member) private returns (uint value) {  
        uint memberUnits = mapEraDay_MemberUnits[_era][_day][_member]; // Get Member Units
        
        if (memberUnits != 0) {
            value = getEmissionShare(_era, _day, _member);          // Get the emission Share for Member
            mapEraDay_MemberUnits[_era][_day][_member] = 0;  // Set to 0 since it will be withdrawn
            mapEraDay_UnitsRemaining[_era][_day] = mapEraDay_UnitsRemaining[_era][_day].sub(memberUnits);  // Decrement Member Units
            mapEraDay_EmissionRemaining[_era][_day] = mapEraDay_EmissionRemaining[_era][_day].sub(value);  // Decrement emission
            totalEmitted += value;                                                 // Add to Total Emitted
            uint256 v_value = value * 3 / 10;                                 // Transfer 30%, lock the rest in vesting contract             
            mainToken.transfer(_member, v_value);                      // ERC20 transfer function
            vestLock.vest(_member, value - v_value, 0);
            emit Withdrawal(msg.sender, _member, _era, _day, value, mapEraDay_EmissionRemaining[_era][_day]);
        }
    }
</code>

