## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [The check for value transfer success is made after the return statement in  _withdrawFromYieldPool of LidoVault](https://github.com/code-423n4/2022-05-sturdy-findings/issues/157) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/LidoVault.sol#L142


# Vulnerability details

## Impact
Users can lose their funds 
## Proof of Concept
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/LidoVault.sol#L142

The code checks transaction success after returning the transfer value and finishing execution. If the call fails the transaction won't revert since  require(sent, Errors.VT_COLLATERAL_WITHDRAW_INVALID); won't execute.

Users will have withdrawed without getting their funds back.


## Recommended Mitigation Steps
Return the function after the success check

