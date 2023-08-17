## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report
- H-02

# [Incorrect output amount calculation for Trader Joe V1 pools](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/345) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L891
https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L896


# Vulnerability details

## Impact
Output amount is calculated incorrectly for a Trader Joe V1 pool when swapping tokens across multiple pools and some of the pools in the chain are V1 ones. Calculated amounts will always be smaller than expected ones, which will always affect chained swaps that include V1 pools.
## Proof of Concept
[LBRouter](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L21) is a high-level contract that serves as the main contract users will interact with. The contract implements a lot of security checks and helper functions that make usage of LBPair contracts easier and more user-friendly. Some examples of such functions:
- [swapExactTokensForTokensSupportingFeeOnTransferTokens](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L531), which makes chained swaps (i.e. swaps between tokens that don't have a pair) of tokens implementing fee on transfer (i.e. there's fee reduced from every transferred amount);
- [swapExactTokensForAVAXSupportingFeeOnTransferTokens](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L561), which is the variation of the above function which takes AVAX as the output token;
- [swapExactAVAXForTokensSupportingFeeOnTransferTokens](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L594), which is the variation of the previous function which takes AVA as the input token.

Under the hood, these three functions call [_swapSupportingFeeOnTransferTokens](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L864), which is the function that actually performs swaps. The function supports both Trader Joe V1 and V2 pools: when `_binStep` is 0 (which is never true in V2 pools), it's assumed that the current pool is a V1 one. For V1 pools, the function calculates output amounts based on pools' reserves and balances:
```solidity
if (_binStep == 0) {
    (uint256 _reserve0, uint256 _reserve1, ) = IJoePair(_pair).getReserves();
    if (_token < _tokenNext) {
        uint256 _balance = _token.balanceOf(_pair);
        uint256 _amountOut = (_reserve1 * (_balance - _reserve0) * 997) / (_balance * 1_000);

        IJoePair(_pair).swap(0, _amountOut, _recipient, "");
    } else {
        uint256 _balance = _token.balanceOf(_pair);
        uint256 _amountOut = (_reserve0 * (_balance - _reserve1) * 997) / (_balance * 1_000);

        IJoePair(_pair).swap(_amountOut, 0, _recipient, "");
    }
} else {
    ILBPair(_pair).swap(_tokenNext == ILBPair(_pair).tokenY(), _recipient);
}
```
However, these calculations are incorrect. Here's the difference:
```diff
@@ -888,12 +888,14 @@ contract LBRouter is ILBRouter {
                     (uint256 _reserve0, uint256 _reserve1, ) = IJoePair(_pair).getReserves();
                     if (_token < _tokenNext) {
                         uint256 _balance = _token.balanceOf(_pair);
-                        uint256 _amountOut = (_reserve1 * (_balance - _reserve0) * 997) / (_balance * 1_000);
+                        uint256 amountInWithFee = (_balance - _reserve0) * 997;
+                        uint256 _amountOut = (_reserve1 * amountInWithFee) / (_reserve0 * 1_000 + amountInWithFee);

                         IJoePair(_pair).swap(0, _amountOut, _recipient, "");
                     } else {
                         uint256 _balance = _token.balanceOf(_pair);
-                        uint256 _amountOut = (_reserve0 * (_balance - _reserve1) * 997) / (_balance * 1_000);
+                        uint256 amountInWithFee = (_balance - _reserve1) * 997;
+                        uint256 _amountOut = (_reserve0 * amountInWithFee) / (_reserve1 * 1_000 + amountInWithFee);

                         IJoePair(_pair).swap(_amountOut, 0, _recipient, "");
                     }
```

These calculations are implemented correctly in [JoeLibrary.getAmountOut](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/libraries/JoeLibrary.sol#L30-L41), which is used in [LBQuoter](https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBQuoter.sol#L83).  Also it's used in Trader Joe V1 to calculate output amounts in similar functions:
- https://github.com/traderjoe-xyz/joe-core/blob/main/contracts/traderjoe/JoeRouter02.sol#L375

```solidity
// test/audit/RouterMath2.t.sol
// SPDX-License-Identifier: UNLICENSED

pragma solidity ^0.8.7;

import "../TestHelper.sol";

import "../../src/LBRouter.sol";
import "../../src/interfaces/IJoePair.sol";

contract RouterMath2Test is TestHelper {
    IERC20 internal token;
    uint256 internal actualAmountOut;

    function setUp() public {
        token = new ERC20MockDecimals(18);
        ERC20MockDecimals(address(token)).mint(address(this), 100e18);

        router = new LBRouter(
            ILBFactory(address(0x00)),
            IJoeFactory(address(this)),
            IWAVAX(address(0x02))
        );
    }

    // Imitates V1 factory.
    function getPair(address, /*tokenX*/ address /*tokenY*/ ) public view returns (address) {
        return address(this);
    }

    // Imitates V1 pool.
    function getReserves() public pure returns (uint112, uint112, uint32) {
        return (1e18, 1e18, 0);
    }

    // Imitates V1 pool.
    function balanceOf(address /*acc*/) public pure returns (uint256) {
        return 0.0001e18;
    }

    // Imitates V1 pool.
    function swap(uint256 amount0, uint256 amount1, address to, bytes memory data) public {
        actualAmountOut = amount0 == 0 ? amount1 : amount0;
    }

    function testScenario() public {
        // Setting up a swap via one V1 pool.
        uint256[] memory steps = new uint256[](1);
        steps[0] = 0;

        IERC20[] memory path = new IERC20[](2);
        path[0] = IERC20(address(token));
        path[1] = IERC20(address(this));

        uint256 amountIn = 0.0001e18;

        token.approve(address(router), 1e18);
        router.swapExactTokensForTokensSupportingFeeOnTransferTokens(
            amountIn, 0, steps, path, address(this), block.timestamp + 1000
        );
        // This amount was calculated incorrectly.
        assertEq(actualAmountOut, 987030000000000000); // Equals to 989970211528238869 when fixed.


        address _pair = address(this);
        uint256 expectedAmountOut;

        // Reproduce the calculations using JoeLibrary.getAmountIn. This piece:
        // https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L888-L899
        (uint256 _reserve0, uint256 _reserve1, ) = IJoePair(_pair).getReserves();
        if (address(token) < address(this)) {
            uint256 _balance = token.balanceOf(_pair);
            expectedAmountOut = JoeLibrary.getAmountOut(_balance - _reserve0, _reserve0, _reserve1);
        } else {
            uint256 _balance = token.balanceOf(_pair);
            expectedAmountOut = JoeLibrary.getAmountOut(_balance - _reserve1, _reserve1, _reserve0);
        }

        // This is the correct amount.
        assertEq(expectedAmountOut, 989970211528238869);

        // The wrong amount is smaller than the expected one.
        assertEq(expectedAmountOut - actualAmountOut, 2940211528238869);
    }
}
```
## Tools Used
Manual review.
## Recommended Mitigation Steps
Consider using the `JoeLibrary.getAmountOut` function in the `_swapSupportingFeeOnTransferTokens` function of `LBRouter` when computing output amounts for V1 pools.