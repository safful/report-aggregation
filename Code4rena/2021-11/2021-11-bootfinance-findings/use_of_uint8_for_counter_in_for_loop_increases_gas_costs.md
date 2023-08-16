## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use of uint8 for counter in for loop increases gas costs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/175) 

# Handle

pants


# Vulnerability details

On L158 of swap.sol, you use a uint8 as the for loop variable:

Due to how the EVM natively works on 256 numbers, using a 8 bit number here introduces additional costs as the EVM has to properly enforce the limits of this smaller type.

See the warning at this link: https://docs.soliditylang.org/en/v0.8.0/internals/layout_in_storage.html#layout-of-state-variables-in-storage

