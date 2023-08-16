## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimziation: Unnecessary pairBalance call](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/310) 

# Handle

gzeon


# Vulnerability details

## Impact
If `msg.sender == issuer`, we don't need to call `pairBalance(msg.sender)`
https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L447
```
        uint256 balance = pairBalance(msg.sender);
        user.hasWithdrawnPair = true;

        if (msg.sender == issuer) {
            balance = lpSupply / 2;

            emit IssuerLiquidityWithdrawn(msg.sender, address(pair), balance);

            if (tokenReserve > 0) {
                uint256 amount = tokenReserve;
                tokenReserve = 0;
                token.transfer(msg.sender, amount);
            }
        } else {
            emit UserLiquidityWithdrawn(msg.sender, address(pair), balance);
        }
```
to
```
        uint256 balance;
        user.hasWithdrawnPair = true;

        if (msg.sender == issuer) {
            balance = lpSupply / 2;

            emit IssuerLiquidityWithdrawn(msg.sender, address(pair), balance);

            if (tokenReserve > 0) {
                uint256 amount = tokenReserve;
                tokenReserve = 0;
                token.transfer(msg.sender, amount);
            }
        } else {
            balance = pairBalance(msg.sender);
            emit UserLiquidityWithdrawn(msg.sender, address(pair), balance);
        }
```

