## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Redeem Sense can be bricked](https://github.com/code-423n4/2022-06-illuminate-findings/issues/244) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/main/redeemer/Redeemer.sol#L262


# Vulnerability details

Sense's `redeem` can be totally DOSd due to user supplied input.

## Impact
Using this attack, Sense market can not be redeemed.

## Proof of Concept
[This](https://github.com/code-423n4/2022-06-illuminate/blob/main/redeemer/Redeemer.sol#L253:#L262) is how Sense market is being redeemed:
```
        IERC20 token = IERC20(IMarketPlace(marketPlace).markets(u, m, p));
        uint256 amount = token.balanceOf(lender);
        Safe.transferFrom(token, lender, address(this), amount);
        ISense(d).redeem(o, m, amount);
```
The problem is that `d` is user supplied input and the function only tries to redeem the amount that was transferred from Lender.

A user can supply malicious `d` contract which does nothing on `redeem(o, m, amount)`.
The user will then call Redeemer's `redeem` with his malicious contract.
Redeemer will transfer all the prinicipal from Lender to itself, will call `d` (noop), and finish.
Sense market has not been redeemed.

Now if somebody tries to call Sense market's `redeem` again, the `amount` variable will be 0, and Redeemer will try to redeem 0 from Sense.

All the original principal is locked and lost in the contract,
like tears in rain.

## Recommended Mitigation Steps
I think you should either use a whitelisted Sense address, or send to `ISense(d).redeem` Redeemer's whole principal balance.

