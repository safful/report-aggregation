## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Struct packing](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/12) 

# Handle

robee


# Vulnerability details

In ITwabRewards.sol, Promotion is optimized to: 4 slots from: 5 slots.
The new order of types (you choose the actual variables):
        1. IERC20
        2. uint216
        3. uint32
        4. uint8
        5. address
        6. uint32
        7. address


