## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-04-axelar-findings/issues/30) 

1. Unmatch comment with actual code

https://github.com/code-423n4/2022-04-axelar/blob/main/src/ERC20.sol#L48

its said that :
    ``` * All three of these values are immutable``

but in actual code`name` and `symbol` arent immutable value. so it would be missleading information on comment sectioin.


##Tool Used 
Manual Review, Remix

##Recommended Mitigation
Change it or remove it

