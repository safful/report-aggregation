## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Unnecessary checked arithmetic when no overflow/underflow possible](https://github.com/code-423n4/2022-01-insure-findings/issues/66) 

# Handle

Dravee


# Vulnerability details

## Impact 
Increased gas cost. 

## Proof of Concept 
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers.

When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation, or the operation doesn't depend on user input), some gas can be saved by using an `unchecked` block.

https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic

These lines are the obvious ones that can't underflow or overflow (operations on constants or checks already made before the operations with `require` statements or `if` statements):
``` 
PremiumModels\BondingPremium.sol:47:        T_1 = 1000000 * DECIMAL;
PremiumModels\BondingPremium.sol:130:        uint256 u1 = BASE - ((_lockedAmount * BASE) / _totalLiquidity); //util rate before. 1000000 = 100.000%
PremiumModels\BondingPremium.sol:132:            (((_lockedAmount + _amount) * BASE) / _totalLiquidity); //util rate after. 1000000 = 100.000%
IndexTemplate.sol:292:                        uint256 _lockedCredit = _allocated - _availableBalance;
PoolTemplate.sol:942:            return a - b;
IndexTemplate.sol:308:                    _retVal = _totalLiquidity - _necessaryAmount;
IndexTemplate.sol:393:                    uint256 _decrease = _current - _target;
IndexTemplate.sol:399:                    uint256 _allocate = _target - _current;
IndexTemplate.sol:441:                _shortage = _amount - _value;
InsureDAOERC20.sol:255:        _balances[sender] = senderBalance - amount;
InsureDAOERC20.sol:303:        _balances[account] = accountBalance - amount;
Vault.sol:165:            uint256 _shortage = _amount - available();
Vault.sol:310:            uint256 _shortage = _retVal - available();
``` 

## Tools Used 
VS Code 

## Recommended Mitigation Steps
Uncheck arithmetic operations when the risk of underflow or overflow is already contained.

