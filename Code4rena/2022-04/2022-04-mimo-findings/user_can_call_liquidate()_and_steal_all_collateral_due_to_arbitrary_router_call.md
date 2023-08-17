## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [User can call liquidate() and steal all collateral due to arbitrary router call](https://github.com/code-423n4/2022-04-mimo-findings/issues/83) 

# Lines of code

https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/liquidityMining/v2/PARMinerV2.sol#L126
https://github.com/Uniswap/v2-periphery/blob/2efa12e0f2d808d9b49737927f0e416fafa5af68/contracts/UniswapV2Router02.sol#L299
https://github.com/Uniswap/solidity-lib/blob/c01640b0f0f1d8a85cba8de378cc48469fcfd9a6/contracts/libraries/TransferHelper.sol#L47-L50


# Vulnerability details

## Impact
A malicious user is able to steal all collateral of an unhealthy position in `PARMinerV2.sol`. The code for the `liquidate()` function is written so that the following steps are followed:

- User calls `PARMinerV2.liquidate()`
- PARMinerV2 performs the liquidation with `_a.parallel().core().liquidatePartial()`
- PARMinerV2 receives the liquidated collateral
- An arbitrary router function is called to swap the collateral to PAR
- Finally, `PARMinerV2.liquidate()` checks that PARMinerV2's PAR balance is higher than the balance at the beginning of the function call.

The exploit occurs with the arbitrary router call. The malicious user is able to supply the `dexTxnData` parameter which dictates the function call to the router. If the user supplied a function such as UniswapV2Router's `swapExactTokenForETH()`, then control flow will be given to the user, allowing them to perform the exploit.

Note: The Mimo developers have stated that the routers used by the protocol will be DEX Aggregators such as 1inch and Paraswap, but this submission will be referring to UniswapV2Router for simplicity. It can be assumed that the dex aggregators currently allow swapping tokens for ETH.

Continuing the exploit, once the attacker has gained control due to the ETH transfer, they are able to swap the ETH for PAR. Finally, they deposit the PAR with `PARMinerV2.deposit()`. This will cause the final check of `liquidate()` to pass because PARMinerV2's PAR balance will be larger than the start of the liquidation call.

The attacker is able to steal all collateral from every unhealthy position that they liquidate. In the most extreme case, the attacker is able to open their own risky positions with the hope that the position becomes unhealthy. They will borrow the PAR and then liquidate themselves to take back the collateral. Thus effectively stealing PAR.

## Proof of Concept
Steps for exploit:

- Attacker monitors unhealthy positions. Finds a position to liquidate.
- Attacker calls `PARMinerV2.liquidate()`
- Position liquidated. Collateral transferred back to `PARMinerV2`
- In the `liquidate()` function, attacker supplies bytes for `UniswapV2Router.swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)`. For `to`, they supply the attacker contract.
- `swapExactTokensForETH()` firstly swaps the collateral for ETH and then transfers the ETH to the user with `TransferHelper.safeTransferETH(to, amounts[amounts.length - 1]);`
- `TransferHelper.safeTransferETH()` contains a call to the receiver via `(bool success, ) = to.call{value: value}(new bytes(0));`
- Therefore, the attacker contract will indeed gain control of execution.

The attacker contract will then perform the following steps:

- Swap the received ETH to PAR.
- Deposit the PAR in `PARMinerV2`
- Withdraw the deposited PAR.

## Tools Used
Static review.

## Recommended Mitigation Steps
The arbitrary call to the router contracts is risky because of the various functions that they can contain. Perhaps a solution is to only allow certain calls such as swapping tokens to tokens, not ETH. This would require frequently updated knowledge of the router's functions, though would be beneficial for security. 

Also, adding a check that the `_totalStake` variable has not increased during the liquidation call will mitigate the risk of the attacker depositing the PAR to increase the contract's balance. The attacker would have no option but to transfer the PAR to PARMinerV2 as is intended.

