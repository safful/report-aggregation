## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Tempus lend method wrongly calculates amount of iPT tokens to mint](https://github.com/code-423n4/2022-06-illuminate-findings/issues/222) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L465:#L466


# Vulnerability details

The Tempus `lend` method calculates the amount of tokens to mint as `amountReturnedFromTempus - lenderBalanceOfMetaPrincipalToken`.
This seems wrong as there's no connection between the two items. Tempus has no relation to the iPT token.

## Impact
Wrong amount of iPT will be minted to the user.
If the Lender contract has iPT balance, the function will revert, otherwise, user will get minted 0 iPT tokes.

## Proof of Concept
[This](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L465:#L469) is how the `lend` method calculates the amount of iPT tokens to mint:
```
        uint256 returned = ITempus(tempusAddr).depositAndFix(Any(x), Any(t), a - fee, true, r, d) -
            illuminateToken.balanceOf(address(this));
        illuminateToken.mint(msg.sender, returned);
```
The Tempus `depositAndFix` method [does not return](https://etherscan.io/address/0xdB5fD0678eED82246b599da6BC36B56157E4beD8#code#F1#L127) anything.
Therefore this calculation will revert if `illuminateToken.balanceOf(address(this)) > 0`, or will return 0 if the balance is 0.

[Note: there's another issue here where the depositAndFix sends wrong parameters - I will submit it in another issue.]

## Recommended Mitigation Steps
I believe that what you intended to do is to check how many Tempus principal tokens the contract received.
So you need to check Lender's `x.tempusPool().principalShare()` before and after the swap, and the delta is the amount received.

