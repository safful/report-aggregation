## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [WETH.allowance() returns wrong result.](https://github.com/code-423n4/2022-06-canto-findings/issues/218) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/WETH.sol#L104


# Vulnerability details

## Impact
WETH.allowance() returns wrong result.
I can't find other contracts that use this function but WETH.sol is a base contract and it should be fixed properly.


## Proof of Concept
In this function, the "return" keyword is missing and it will always output 0 in this case.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
L104 should be changed like below.
```
return _allowance[owner][spender];
```

