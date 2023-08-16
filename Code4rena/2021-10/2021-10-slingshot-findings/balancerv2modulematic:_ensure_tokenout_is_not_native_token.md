## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [BalancerV2ModuleMatic: Ensure tokenOut is not native token](https://github.com/code-423n4/2021-10-slingshot-findings/issues/38) 

# Handle

hickuphh3


# Vulnerability details

## Impact

The executioner is designed to handle only ERC20-ERC20 token trades by modules. The balancer V2 vault is able to [automatically unwrap the wrapped native token](https://dev.balancer.fi/helpers/using-native-eth#overview). Hence, it is recommended to ensure that the `tokenOut` parameter passed into the `swap()` function is not the sentinel value.

The [sentinel value used is the null address.](https://dev.balancer.fi/helpers/using-native-eth#sentinel-value)

## Recommended Mitigation Steps

Consider adding the following check in the function.

`require(tokenOut != address(0), 'native token swap not supported');`

