## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [_withdrawShare Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/26) 

# Handle

ye0lde


# Vulnerability details

## Impact

The "else if", the second call to _processWithdrawal, and the return statement can be eliminated to reduce gas and improve code clarity.

## Proof of Concept

The _withdrawShare function is here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L201-L210

## Tools Used
Visual Studio Code

## Recommended Mitigation Steps
I recommend rewriting it as follows:
<code>
 function _withdrawShare (uint _era, uint _day, address _member) private returns (uint value) {
	_updateEmission();
	
	if (_era < currentEra ||                                                          // Allow if in previous Era                                                                     
	   (_era == currentEra && _day < currentDay)) {                // or current Era and previous day 
		value = _processWithdrawal(_era, _day, _member);   // Process Withdrawal    
	}  
}
</code>

