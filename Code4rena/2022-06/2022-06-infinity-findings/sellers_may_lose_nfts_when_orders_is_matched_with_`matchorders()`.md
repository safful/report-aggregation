## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [Sellers may lose NFTs when orders is matched with `matchOrders()`](https://github.com/code-423n4/2022-06-infinity-findings/issues/164) 

# Lines of code

https://github.com/infinitydotxyz/exchange-contracts-v2/blob/c51b7e8af6f95cc0a3b5489369cbc7cee060434b/contracts/core/InfinityOrderBookComplication.sol#L205


# Vulnerability details

## Impact

Function `matchOrders` uses custom constraints to make the matching more flexible, allow seller/buyer to specify maximum/minimum number of NFTs they want to sell/buy. This function first does some checks and then execute matching.

But in [function](https://github.com/infinitydotxyz/exchange-contracts-v2/blob/c51b7e8af6f95cc0a3b5489369cbc7cee060434b/contracts/core/InfinityOrderBookComplication.sol#L192) `areNumItemsValid()`, there is a wrong checking will lead to wrong logic in `matchOrders()` function.

Instead of checking if `numConstructedItems <= sell.constraints[0]` or not, function `areNumItemsValid()` check if `buy.constraints[0] <= sell.constraints[0]`. It will lead to the scenario that `numConstructedItems > sell.constraints[0]` and make the seller sell more number of nfts than he/she allow. 


## Proof of concept 

Consider the scenario
1. Alice create a sell order to sell maximum 2 in her 3 BAYC with ids `[1, 2, 3]` 
2. Bob create a buy order to buy mimimum any 2 BAYC with id in list `[1, 2, 3]`
3. Match executor call `matchOrders()` to match Alice's order and Bob's one with parameter `constructs = [1, 2, 3]` 
4. Function `matchOrders` will transfer all NFT in `construct` list (3 NFTs `1, 2, 3`) from seller to buyer even though seller only want to sell maximum 2 NFTs.

For more information, please check this PoC. 
https://gist.github.com/minhquanym/a95c8652de8431c5d1d24aa4076a1878
    
    
## Tools Used

hardhat, chai
    
## Recommended Mitigation Steps

Replace check `buy.constraints[0] <= sell.constraints[0]` with `numConstructedItems <= sell.constraints[0]`

