## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`Timelock` Struct Packing in `Vesting.sol`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/307) 

# Handle

leastwood


# Vulnerability details

## Impact

The `Timelock` struct is used to reference the `releaseTimestamp` and vested `amount` for each vesting. These values can likely be safely stored as `uint64` and `uint192` values respectively, enabling the struct to be stored within a single slot instead of two slots.

## Proof of Concept

https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L32-L35

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider updating `releaseTimestamp` to `uint64` and `amount` to `uint192` within the `Timelock` struct. It might be worthwhile performing sanity checks when storing these values by using OpenZeppelin's safe math and safe cast libraries.

