## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Optimize structs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/290) 

# Handle

pauliax


# Vulnerability details

## Impact
Member total_tokens in both structs Airdrop and Investors is practically not used and is a duplicate of the amount so you can remove it to save some storage. Also, gas efficiency can be improved by tightly packing the struct. Struct variables are stored in 32 bytes each so you can group smaller types to occupy less storage, e.g. airdropBalances which are later translated to the amount in Airdrop struct (10**18) can be stored in a smaller version of uint as we know all the exact values at compile time.


