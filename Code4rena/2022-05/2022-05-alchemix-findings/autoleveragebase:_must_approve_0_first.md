## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [AutoleverageBase: Must approve 0 first](https://github.com/code-423n4/2022-05-alchemix-findings/issues/144) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/71abbe683dfd5c0686b7e594fb4f78a14b668d8b/contracts-full/AutoleverageBase.sol#L61-L63


# Vulnerability details

## Impact
Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved.
## Proof of Concept
https://github.com/code-423n4/2022-05-alchemix/blob/71abbe683dfd5c0686b7e594fb4f78a14b668d8b/contracts-full/AutoleverageBase.sol#L61-L63
https://github.com/code-423n4/2022-05-alchemix/blob/71abbe683dfd5c0686b7e594fb4f78a14b668d8b/contracts-full/AutoleverageBase.sol#L147-L147
https://github.com/code-423n4/2022-05-alchemix/blob/71abbe683dfd5c0686b7e594fb4f78a14b668d8b/contracts-full/AutoleverageBase.sol#L178-L179
## Tools Used
None
## Recommended Mitigation Steps
```
    function approve(address token, address spender) internal {
    +  IERC20(token).approve(spender, 0);
        IERC20(token).approve(spender, type(uint256).max);
    }
```

