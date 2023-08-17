## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report
- H-04

# [Wrong calculation in function `LBRouter._getAmountsIn` make user lose a lot of tokens when swap through JoePair (most of them will gifted to JoePair freely)](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/400) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L725


# Vulnerability details


## Vulnerable detail 
Function `LBRouter._getAmountsIn` is a helper function to return the amounts in with given `amountOut`. This function will check the pair of `_token` and `_tokenNext` is `JoePair` or `LBPair` using `_binStep`.
* If `_binStep == 0`, it will be a `JoePair` otherwise it will be an `LBPair`.
```solidity=
if (_binStep == 0) {
    (uint256 _reserveIn, uint256 _reserveOut, ) = IJoePair(_pair).getReserves();
    if (_token > _tokenPath[i]) {
        (_reserveIn, _reserveOut) = (_reserveOut, _reserveIn);
    }


    uint256 amountOut_ = amountsIn[i];
    // Legacy uniswap way of rounding
    amountsIn[i - 1] = (_reserveIn * amountOut_ * 1_000) / (_reserveOut - amountOut_ * 997) + 1;
} else {
    (amountsIn[i - 1], ) = getSwapIn(ILBPair(_pair), amountsIn[i], ILBPair(_pair).tokenX() == _token);
}
```
As we can see when `_binStep == 0` and `_token < _tokenPath[i]` (in another word  we swap through `JoePair` and pair's`token0` is `_token` and `token1` is `_tokenPath[i]`), it will 
1. Get the reserve of pair (`reserveIn`, `reserveOut`) 
2. Calculate the `_amountIn` by using the formula 
```
amountsIn[i - 1] = (_reserveIn * amountOut_ * 1_000) / (_reserveOut - amountOut_ * 997) + 1
```

But unfortunately the denominator `_reserveOut - amountOut_ * 997` seem incorrect. It should be `(_reserveOut - amountOut_) * 997`. 
We will do some math calculations here to prove the expression above is wrong. 

**Input:** 
* `_reserveIn (rIn)`: reserve of `_token` in pair 
* `_reserveOut (rOut)`: reserve of `_tokenPath[i]` in pair 
* `amountOut_`: the amount of `_tokenPath` the user wants to gain 
 
**Output:** 
* `rAmountIn`: the actual amount of `_token` we need to transfer to the pair. 

**Generate Formula** 
Cause `JoePair` [takes 0.3%](https://help.traderjoexyz.com/en/welcome/faq-and-help/general-faq#what-are-trader-swap-joe-fees) of `amountIn` as fee, we get 
* `amountInDeductFee = amountIn' * 0.997`

Following the [constant product formula](https://docs.uniswap.org/protocol/V2/concepts/protocol-overview/glossary#constant-product-formula), we have 
```
    rIn * rOut = (rIn + amountInDeductFee) * (rOut - amountOut_)
==> rIn + amountInDeductFee = rIn * rOut / (rOut - amountOut_) + 1
<=> amountInDeductFee = (rIn * rOut) / (rOut - amountOut_) - rIn + 1
<=> rAmountIn * 0.997 = rIn * amountOut / (rOut - amountOut_) + 1
<=> rAmountIn = (rIn * amountOut * 1000) / ((rOut - amountOut_) * 997) + 1
<=> 
```

As we can see `rAmountIn` is different from `amountsIn[i - 1]`, the denominator of `rAmountIn` is `(rOut - amountOut_) * 997` when the denominator of `amountsIn[i - 1]` is `_reserveOut - amountOut_ * 997` (Missing one bracket)

## Impact
**Loss of fund: User will send a lot of tokenIn (much more than expected) but just gain exact amountOut in return.** 

Let dive in the function `swapTokensForExactTokens()` to figure out why this scenario happens. I will assume I just swap through only one pool from `JoePair` and 0 pool from `LBPair`. 
* Firstly function will get the list `amountsIn` from function `_getAmountsIn`. So `amountsIn` will be [`incorrectAmountIn`, `userDesireAmount`]. 
    ```solidity=        
    // url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L440
    amountsIn = _getAmountsIn(_pairBinSteps, _pairs, _tokenPath, _amountOut);
    ``` 
* Then it transfers `incorrectAmountIn` to `_pairs[0]` to prepare for the swap. 
    ```solidity=
    // url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L444
    _tokenPath[0].safeTransferFrom(msg.sender, _pairs[0], amountsIn[0]);
    ```  
* Finally it calls function `_swapTokensForExactToken` to execute the swap. 
    ```solidity=
    // url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L446    
    uint256 _amountOutReal = _swapTokensForExactTokens(_pairs, _pairBinSteps, _tokenPath, amountsIn, _to);
    ```
    In this step it will reach to [line 841](https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L841) which will set the expected `amountOut = amountsIn[i+1] = amountsIn[1] = userDesireAmount`.
    ```solidity=
    // url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L841
    amountOut = _amountsIn[i + 1];
    ```
    So after calling `IJoePair(_pair).swap()`, the user just gets exactly `amountOut` and wastes a lot of tokenIn that (s)he transfers to the pool. 


## Proof of concept 
Here is our test script to describe the impacts 
* https://gist.github.com/huuducst/6e34a7bdf37bb29f4b84d2faead94dc4

You can place this file into `/test` folder and run it using 
```bash=
forge test --match-test testBugSwapJoeV1PairWithLBRouter --fork-url https://rpc.ankr.com/avalanche --fork-block-number 21437560 -vv
```

Explanation of test script: (For more detail u can read the comments from test script above)
1. Firstly we get the Joe v1 pair WAVAX/USDC from JoeFactory.
2. At the forked block, price `WAVAX/USDC` was around 15.57. We try to use LBRouter function `swapTokensForExactTokens` to swap 10$ WAVAX (10e18 wei) to 1$ USDC (1e6 wei). But it reverts with the error `LBRouter__MaxAmountInExceeded`.
But when we swap directly to JoePair, it swap successfully 10$ AVAX (10e18 wei) to 155$ USDC (155e6 wei).
3. We use LBRouter function `swapTokensForExactTokens` again with very large `amountInMax` to swap 1$ USDC (1e6 wei). It swaps successfully but needs to pay a very large amount WAVAX (much more than price).

## Tools Used
Foundry 
 
## Recommended Mitigation Steps
Modify function `LBRouter._getAmountsIn` as follow
```solidity=
// url = https://github.com/code-423n4/2022-10-traderjoe/blob/79f25d48b907f9d0379dd803fc2abc9c5f57db93/src/LBRouter.sol#L717-L728
if (_binStep == 0) {
    (uint256 _reserveIn, uint256 _reserveOut, ) = IJoePair(_pair).getReserves();
    if (_token > _tokenPath[i]) {
        (_reserveIn, _reserveOut) = (_reserveOut, _reserveIn);
    }


    uint256 amountOut_ = amountsIn[i];
    // Legacy uniswap way of rounding
    // Fix here 
    amountsIn[i - 1] = (_reserveIn * amountOut_ * 1_000) / ((_reserveOut - amountOut_) * 997) + 1;
} else {
    (amountsIn[i - 1], ) = getSwapIn(ILBPair(_pair), amountsIn[i], ILBPair(_pair).tokenX() == _token);
}
```
