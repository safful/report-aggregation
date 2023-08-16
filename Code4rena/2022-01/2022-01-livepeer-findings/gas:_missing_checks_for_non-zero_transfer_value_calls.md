## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Missing checks for non-zero transfer value calls](https://github.com/code-423n4/2022-01-livepeer-findings/issues/141) 

# Handle

Dravee


# Vulnerability details

## Impact
Checking non-zero transfer values can avoid an external call to save gas.

## Proof of Concept
Instances missing a non-zero check:
```
arbitrum-lpt-bridge\contracts\L1\gateway\L1LPTGateway.sol:100:            TokenLike(_l1Token).transferFrom(from, l1LPTEscrow, _amount);
arbitrum-lpt-bridge\contracts\L1\gateway\L1LPTGateway.sol:150:            TokenLike(l1Token).transferFrom(l1LPTEscrow, to, amount);
protocol\contracts\token\BridgeMinter.sol:79:        token.transfer(_newMinterAddr, token.balanceOf(address(this)));
protocol\contracts\token\BridgeMinter.sol:109:        token.transfer(l1MigratorAddr, balance); 
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Check if transfer amount > 0 before executing the transfer

