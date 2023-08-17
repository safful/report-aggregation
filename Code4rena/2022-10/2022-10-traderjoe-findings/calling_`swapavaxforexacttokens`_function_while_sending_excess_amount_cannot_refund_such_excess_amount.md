## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- sponsor confirmed
- selected for report
- M-06

# [Calling `swapAVAXForExactTokens` function while sending excess amount cannot refund such excess amount](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/469) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L485-L521


# Vulnerability details

## Impact
When calling the `swapAVAXForExactTokens`  function, `if (msg.value > amountsIn[0]) _safeTransferAVAX(_to, amountsIn[0] - msg.value)` is executed, which is for refunding any excess amount sent in; this is confirmed by this function's comment as well. However, executing `amountsIn[0] - msg.value` will always revert when `msg.value > amountsIn[0]` is true. Developers who has the design of the `swapAVAXForExactTokens` function in mind could develop front-ends and contracts that will send excess amount when calling the `swapAVAXForExactTokens` function. Hence, the users, who rely on these front-ends and contracts for interacting with the `swapAVAXForExactTokens` function will always find such interactions being failed since calling this function with the excess amount will always revert. As a result, the user experience becomes degraded, and the usability of the protocol becomes limited.

https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L485-L521
```solidity
    /// @notice Swaps AVAX for exact tokens while performing safety checks
    /// @dev will refund any excess sent
    ...
    function swapAVAXForExactTokens(
        uint256 _amountOut,
        uint256[] memory _pairBinSteps,
        IERC20[] memory _tokenPath,
        address _to,
        uint256 _deadline
    )
        external
        payable
        override
        ensure(_deadline)
        verifyInputs(_pairBinSteps, _tokenPath)
        returns (uint256[] memory amountsIn)
    {
        ...

        if (msg.value > amountsIn[0]) _safeTransferAVAX(_to, amountsIn[0] - msg.value);
    }
```

## Proof of Concept
Please add the following test in `test\LBRouter.Swaps.t.sol`. This test will pass to demonstrate the described scenario.
```solidity
    function testSwapAVAXForExactTokensIsUnableToRefund() public {
        uint256 amountOut = 1e18;

        (uint256 amountIn, ) = router.getSwapIn(pairWavax, amountOut, false);

        IERC20[] memory tokenList = new IERC20[](2);
        tokenList[0] = wavax;
        tokenList[1] = token6D;
        uint256[] memory pairVersions = new uint256[](1);
        pairVersions[0] = DEFAULT_BIN_STEP;

        vm.deal(DEV, amountIn + 500);

        // Although the swapAVAXForExactTokens function supposes to refund any excess sent,
        //   calling it reverts when sending more than amountIn
        //   because executing _safeTransferAVAX(_to, amountsIn[0] - msg.value) results in arithmetic underflow
        vm.expectRevert(stdError.arithmeticError);
        router.swapAVAXForExactTokens{value: amountIn + 1}(amountOut, pairVersions, tokenList, DEV, block.timestamp);
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-10-traderjoe/blob/main/src/LBRouter.sol#L520 can be updated to the following code.
```solidity
        if (msg.value > amountsIn[0]) _safeTransferAVAX(_to, msg.value - amountsIn[0]);
```