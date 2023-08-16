## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-03-biconomy-findings/issues/5) 

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/ExecutorManager.sol#L10
The executorstatus map is not really helpful in any business logic. If the executor is present in executors array then the status is true or else false. 
Once the executor is added, it cannot be added again even though the status is set to FALSE.

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/ExecutorManager.sol#L53
The function removeExecutor is just setting the status to false.

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/ExecutorManager.sol#L55
Check if the executor is present and executorStatus is not already set to false.


https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/token/svg-helpers/SvgHelperBase.sol#L126
Both inpurt parameters are not used in the function. Remove these.


https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L169
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L330
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L333
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityPool.sol#L107
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityPool.sol#L113
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityPool.sol#L123
If function is not called from inside the contract, better to make it external

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/LiquidityFarming.sol#L211
NFTInfo storage nft = nftInfo[_nftId];
require(!nft.isStaked, "ERR__NFT_ALREADY_STAKED");
Move above code immidiately after the require(lpToken.isApprovedForAll...). If NFT is not approved, then no need to get token metadata
