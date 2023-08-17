## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Leak of Value in `yield` function, slippage check is not effective](https://github.com/code-423n4/2022-06-illuminate-findings/issues/289) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L641-L654


# Vulnerability details

The function `yield` is using the input from `sellBasePreview` and then using it.

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/lender/Lender.sol#L641-L654

```solidity
    function yield(
        address u,
        address y,
        uint256 a,
        address r
    ) internal returns (uint256) {
        // preview exact swap slippage on yield
        uint128 returned = IYield(y).sellBasePreview(Cast.u128(a));

        // send the remaing amount to the given yield pool
        Safe.transfer(IERC20(u), y, a);

        // lend out the remaining tokens in the yield pool
        IYield(y).sellBase(r, returned);
```

The output of `sellBasePreview` is meant to be used off-chain to avoid front-running and price changes, additionally no validation is performed on this value (is it zero, is it less than 95% of amount) meaning the check is equivalent to setting `returned = 0`

I'd recommend to add checks, or ideally have a trusted keeper bulk `sellBase` with an additional slippage check as the function parameter

