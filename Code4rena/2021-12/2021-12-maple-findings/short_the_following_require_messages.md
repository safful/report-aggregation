## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Short the following require messages](https://github.com/code-423n4/2021-12-maple-findings/issues/3) 

# Handle

robee


# Vulnerability details

The following require messages are of length more than 32 and we think are short enough to short
them into exactly 32 characters such that it will be placed in one slot of memory and the require 
function will cost less gas. 
The list: 

        Solidity file: SushiswapStrategy.sol, In line 53, Require message length to shorten: 38
        Solidity file: SushiswapStrategy.sol, In line 74, Require message length to shorten: 33
        Solidity file: UniswapV2Strategy.sol, In line 53, Require message length to shorten: 38
        Solidity file: UniswapV2Strategy.sol, In line 74, Require message length to shorten: 33
        Solidity file: MapleProxyFactory.sol, In line 33, Require message length to shorten: 36
        Solidity file: MapleProxyFactory.sol, In line 42, Require message length to shorten: 36

