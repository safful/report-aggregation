## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Short the following require messages](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/7) 

# Handle

robee


# Vulnerability details

The following require messages are of length more than 32 and we think are short enough to short
them into exactly 32 characters such that it will be placed in one slot of memory and the require 
function will cost less gas. 
The list: 

        Solidity file: ActivePool.sol, In line 252, Require message length to shorten: 35, The message: ActivePool: Caller is not whitelist
        Solidity file: BorrowerOperations.sol, In line 215, Require message length to shorten: 39, The message: BOps: colls and amounts length mismatch
        Solidity file: BorrowerOperations.sol, In line 874, Require message length to shorten: 33, The message: BOps: Collateral not in whitelist
        Solidity file: CollSurplusPool.sol, In line 167, Require message length to shorten: 40, The message: CollSurplusPool: Caller is not Whitelist
        Solidity file: DefaultPool.sol, In line 122, Require message length to shorten: 38, The message: DefaultPool: sending collateral failed
        Solidity file: DefaultPool.sol, In line 167, Require message length to shorten: 36, The message: DefaultPool: Caller is not whitelist
        Solidity file: LiquitySafeMath128.sol, In line 10, Require message length to shorten: 37, The message: LiquitySafeMath128: addition overflow
        Solidity file: LiquitySafeMath128.sol, In line 16, Require message length to shorten: 40, The message: LiquitySafeMath128: subtraction overflow
        Solidity file: SafeMath.sol, In line 87, Require message length to shorten: 33, The message: SafeMath: multiplication overflow
        Solidity file: Address.sol, In line 115, Require message length to shorten: 38, The message: Address: insufficient balance for call
        Solidity file: Address.sol, In line 140, Require message length to shorten: 36, The message: Address: static call to non-contract
        Solidity file: SortedTroves.sol, In line 131, Require message length to shorten: 34, The message: SortedTroves: ICR must be positive
        Solidity file: SortedTroves.sol, In line 230, Require message length to shorten: 34, The message: SortedTroves: ICR must be positive
        Solidity file: StabilityPool.sol, In line 1076, Require message length to shorten: 39, The message: StabilityPool: Caller is not ActivePool
        Solidity file: StabilityPool.sol, In line 1095, Require message length to shorten: 40, The message: StabilityPool: User must have no deposit
        Solidity file: StabilityPool.sol, In line 1099, Require message length to shorten: 38, The message: StabilityPool: Amount must be non-zero
        Solidity file: StabilityPool.sol, In line 1138, Require message length to shorten: 36, The message: DefaultPool: Caller is not whitelist
        Solidity file: SortedTrovesTester.sol, In line 23, Require message length to shorten: 34, The message: SortedTroves: ICR must be positive
        Solidity file: TroveManagerLiquidations.sol, In line 180, Require message length to shorten: 34, The message: TroveManager: nothing to liquidate
        Solidity file: TroveManagerRedemptions.sol, In line 520, Require message length to shorten: 34, The message: must be non zero redemption amount
        Solidity file: CommunityIssuance.sol, In line 131, Require message length to shorten: 35, The message: CommunityIssuance: caller is not SP
        Solidity file: YETIToken.sol, In line 198, Require message length to shorten: 36, The message: YETI: transfer from the zero address
        Solidity file: YETIToken.sol, In line 222, Require message length to shorten: 39, The message: YETI: caller must be the SYETI contract
        Solidity file: YUSDToken.sol, In line 294, Require message length to shorten: 37, The message: YUSD: Caller is not the StabilityPool

