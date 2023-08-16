## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Save Gas With The Unchecked Keyword (MochiVault.sol)](https://github.com/code-423n4/2021-10-mochi-findings/issues/82) 

# Handle

ye0lde


# Vulnerability details

## Impact

Redundant arithmetic underflow/overflow checks can be avoided when an underflow/overflow cannot happen.

## Proof of Concept

The "unchecked" keyword can be applied here since there is an "if" statement before to ensure the arithmetic operations, would not cause an integer underflow or overflow.
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/vault/MochiVault.sol#L267

Change the code at 267 to:
<code>
unchecked {
   debts -= _amount;
}
</code>

A similar change can be made here:
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/vault/MochiVault.sol#L269

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Add the "unchecked" keyword as shown above.

