## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Frontrunning in UniswapHandler calls to UniswapV2Router](https://github.com/code-423n4/2021-11-malt-findings/issues/219) 

# Handle

thank_you


# Vulnerability details

## Impact
UniswapHandler utilizes UniswapV2Router to swap, add liquidity, and remove liquidity with the UniswapV2Pair contract. In order to utilize these functionalities, UniswapHandler must call various UniswapV2Router methods.

- addLiquidity
- removeLiquidity
- swapExactTokensForTokens (swaps for both DAI and Malt)

In all three methods, UniswapV2Router requires the callee to provide input arguments that define how much the amount out minimum UniswapHandler will allow for a trade. This argument is designed to prevent slippage and more importantly, sandwich attacks.

UniswapHandler correctly handles price slippage when calling [addLiquidity](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L201). However, that is not the case for [removeLiquidity](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L230) and swapExactTokensForTokens [here](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L148) and [here](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L173). For both methods, 0 is passed in as the amount out minimum allowed for a trade. This allows for anyone watching the mempool to sandwich attack UniswapHandler (or any contract that calls UniswapHandler) in such a way that allows the hacker to profit off of a guaranteed trade.

How does this work? Let's assume UniswapHandler makes a call to [UniswapV2Router#swapExactTokensForTokens](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L148) to trade DAI for Malt. Any hacker who watches the mempool and sees this transaction can immediately buy as much Malt as they want. This raises the price of Malt. Since UniswapHandler is willing to accept any amount out minimum (the number is set to [zero](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L150)), then the UniswapHandler will always trade DAI for Malt. This second transaction raises the price of Malt even further. Finally, the hacker trades their Malt for DAI, receiving a profit due to the artificially inflated price of Malt from the sandwich attack.

It's important to note that anyone has access to the UniswapV2Router contract. There are no known ACL controls on UniswapV2Router. This sandwich attack can impact even the `buyMalt` function.

The following functions when called are vulnerable to frontrunning attacks:

- [UniswapHandler#buyMalt](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L131)
- [UniswapHandler#sellMalt](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L160)
- [UniswapHandler#removeLiquidity](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L221)

And by extension the following contract functions since they also call the UniswapHandler function calls:

- [Bonding#unbondAndBreak](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Bonding.sol#L114)
- [LiquidityExtension#purchaseAndBurn](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/LiquidityExtension.sol#L117)
- [RewardReinvestor#splitReinvest](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/RewardReinvestor.sol#L78)
- [StabilizerNode#stabilize](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L145)
- [SwingTrader#buyMalt](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L50)


## Proof of Concept
Refer to the impact section for affected code and links to the appropriate LoC.

## Tools Used
N/A

## Recommended Mitigation Steps
The UniswapV2Router and UniswapV2Pair contract should allow only the UniswapHandler contract to call either contract. In addition, price slippage checks should be implemented whenever removing liquidity or swapping tokens. This ensures that a frontrunning attack can't occur.


## Anything Else We Should Know
I wish I had more time to work on this bug but unfortunately I have several current clients who require significant time from me. I'm happy to pursue this beyond the initial submission, in particular building a concrete PoC. I think the most important takeaway from this bug find is that anyone can purchase Malt at any time and anyone can manipulate the Malt reserve. This in turn impacts other functionalities that rely on the Malt reserve to make price/token calculations such as exiting an auction early or reinvesting rewards.

