## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [unexpected overflow for FullMath.add512() which can result in irregular behavior](https://github.com/code-423n4/2023-01-timeswap-findings/issues/182) 

# Lines of code

https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-library/src/FullMath.sol#L62


# Vulnerability details

## Impact
The vulnerability originate from insufficient checking in add512 function, where the AddOverflow revert gets bypassed, essentially the function assumes that an overflow only happen if (addendA1 > sum1), where in the case that it's possible for it overflow in the case that addendA1 == sum1, which can be resulted through assigning a value that makes ( lt(sum0, addendA0) == 1 ) <=> sum0 < addendA0, which can only be achieved normally by overflowing the least significant addition. then we can simply break the overflow check by assigning an overflowing values which results in add(addendA1, addendB1) > type(256).max && addendA1 <= sum1, then we will manage to bypass the revert check and overflow the most significant part of add512 values.

the previous attack vector can lead to a manipulation in leverage and deleverage functions, in a way that it would result in more tokens for the user
## Proof of Concept
inputing the following values result in an overflow:
uint256 addendA0 = 1
uint256 addendA1 = 100
uint256 addendB0 = 115792089237316195423570985008687907853269984665640564039457584007913129639935 (uint256 max value)
uint256 addendB1 = 115792089237316195423570985008687907853269984665640564039457584007913129639935 (uint256 max value)

results in:
sum0 = 0
sum1 = 100

the expected behavior is to revert since, A1 + B1 result in a value that overflows, but instead consider it as a valid behavior due to the insufficient checking.

Abstraction:
A1 - A0 
                +
B1 - B0
                =
S1 - S0

S0 = A0 + B0
S1 = A1 + B1 + ( if S0 overflows [+ 1])
ensure A1 <= S1
revert only on A1 > S1

in the case of S0 overflows:
S1 = A1 + B1 + 1

require(A1 <= S1) is not most suited check, due to the fact that in the case of A1 == S1 check, it can still overflows if S1 = A1 + B1 + 1 overflows.
which would bypass A1 > S1 revert check.

the major impact affects the leverage() and deleverage() result in values which are not expected.

## Tools Used

## Recommended Mitigation Steps

add a an equality check for if statement in add512 function.

```
   function add512(uint256 addendA0, uint256 addendA1, uint256 addendB0, uint256 addendB1) public pure returns (uint256 sum0, uint256 sum1) {
        assembly {
            sum0 := add(addendA0, addendB0)
            carry  := lt(sum0, addendA0)
            sum1 := add(add(addendA1, addendB1), carry)
        }
        if (addendA1 > sum1 ||      ((sum1 == addendA1 || sum1 == addendB1) && (carry < addendA0 || carry < addendB0))
) revert AddOverflow(addendA0, addendA1, addendB0, addendB1);
// 
    }
```