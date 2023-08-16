## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Report](https://github.com/code-423n4/2022-02-concur-findings/issues/44) 

# Handle

robee


# Vulnerability details

Title: Unnecessary equals boolean
Severity: GAS


Boolean variables can be checked within conditionals directly without the use of equality operators to true/false.

        VoteProxy.sol, 21: if (auctioneer.isWinningSignature(_hash, _signature) == true) {



Title: State variables that could be set immutable
Severity: GAS

In the following files there are state variables that could be set immutable to save gas. 

        startBlock in MasterChef.sol
        client in Shelter.sol
        masterChef in ConvexStakingWrapper.sol
        endBlock in MasterChef.sol
        rewardNotifier in ConcurRewardPool.sol
        concur in MasterChef.sol



Title: Caching array length can save gas
Severity: GAS


Caching the array length is more gas efficient.
This is because access to a local variable in solidity is more efficient than query storage / calldata / memory.
We recommend to change from:    

    for (uint256 i=0; i<array.length; i++) { ... }

to: 

    uint len = array.length  
    for (uint256 i=0; i<len; i++) { ... }


        ConcurRewardPool.sol, _tokens, 35



Title: Prefix increments are cheaper than postfix increments
Severity: GAS

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        just change to unchecked: MasterChef.sol, _pid, 129
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 219
        change to prefix increment and unchecked: ConvexStakingWrapper.sol, i, 121
        change to prefix increment and unchecked: ConcurRewardPool.sol, i, 35



Title: Unnecessary index init
Severity: GAS


In for loops you initialize the index to start from 0, but it already initialized to 0 in default and this assignment cost gas. 
It is more clear and gas efficient to declare without assigning 0 and will have the same meaning:

        ConcurRewardPool.sol, 35
        ConvexStakingWrapper.sol, 219
        MasterChef.sol, 129
        ConvexStakingWrapper.sol, 121



Title: Internal functions to private
Severity: GAS

The following functions could be set private to save gas and improve code quality:

        EasySign.sol, tryRecover
        ConvexStakingWrapper.sol, _calcRewardIntegral
        ConvexStakingWrapper.sol, _checkpoint
        ConvexStakingWrapper.sol, _getDepositedBalance
        ConvexStakingWrapper.sol, _getTotalSupply
        EasySign.sol, recover



Title: Public functions to external
Severity: GAS

The following functions could be set external to save gas and improve code quality. 
External call cost is less expensive than of public functions. 

        MasterChef.sol, massUpdatePools
        MasterChef.sol, add
        ConvexStakingWrapper.sol, addRewards



Title: Unnecessary default assignment
Severity: GAS


Unnecessary default assignments, you can just declare and it will save gas and have the same meaning.
    

        StakingRewards.sol (L#22) : uint256 public rewardRate = 0;
        ConvexStakingWrapper.sol (L#36) : uint256 public constant CRV_INDEX = 0;
        MasterChef.sol (L#51) : uint public totalAllocPoint = 0;
        StakingRewards.sol (L#21) : uint256 public periodFinish = 0;



Title: Rearrange state variables
Severity: GAS

You can change the order of the storage variables to decrease memory uses.

In USDMPegRecovery.sol,rearranging the storage fields can optimize to: 7 slots from: 8 slots.
The new order of types (you choose the actual variables):
        1. IERC20
        2. IERC20
        3. ICurveMetaPool
        4. uint256
        5. uint256
        6. Liquidity
        7. address
        8. bool




Title: Short the following require messages
Severity: GAS

The following require messages are of length more than 32 and we think are short enough to short
them into exactly 32 characters such that it will be placed in one slot of memory and the require 
function will cost less gas. 
The list: 

        Solidity file: StakingRewards.sol, In line 170, Require message length to shorten: 33, The message: Cannot withdraw the staking token
        Solidity file: MasterChef.sol, In line 210, Require message length to shorten: 35, The message: safeConcurTransfer: transfer failed



Title: Unused imports
Severity: GAS


In the following files there are contract imports that aren't used
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore)

        ConvexStakingWrapper.sol, line 9, import "./external/ConvexInterfaces.sol";
        USDMPegRecovery.sol, line 7, import { ICurveMetaPool } from "./external/CurveInterfaces.sol";



Title: Use != 0 instead of > 0
Severity: GAS


Using != 0 is slightly cheaper than > 0. (see https://github.com/code-423n4/2021-12-maple-findings/issues/75 for similar issue)


        MasterChef.sol, 169: change '_amount > 0' to '_amount != 0'
        ConvexStakingWrapper.sol, 262: change '_amount > 0' to '_amount != 0'
        MasterChef.sol, 194: change '_amount > 0' to '_amount != 0'
        USDMPegRecovery.sol, 74: change 'balance > 0' to 'balance != 0'
        ConvexStakingWrapper.sol, 236: change '_amount > 0' to '_amount != 0'
        StakingRewards.sol, 94: change 'amount > 0' to 'amount != 0'
        StakingRewards.sol, 108: change 'amount > 0' to 'amount != 0'
        ConvexStakingWrapper.sol, 184: change '_supply > 0' to '_supply != 0'



Title: Use unchecked to save gas for certain additive calculations that cannot overflow
Severity: GAS


You can use unchecked in the following calculations since there is no risk to overflow:

        StakingRewards.sol (L#161) - periodFinish = block.timestamp + rewardsDuration;
        Shelter.sol (L#53) - require(activated[_token] != 0 && activated[_token] + GRACE_PERIOD < block.timestamp, "shelter not activated");
        ConvexStakingWrapper.sol (L#279) - return uint64((block.timestamp - VOTECYCLE_START) / 2 weeks) + 1;
        Shelter.sol (L#45) - require(activated[_token] != 0 && activated[_token] + GRACE_PERIOD > block.timestamp, "too late");



Title: Use calldata instead of memory
Severity: GAS


Use calldata instead of memory for function parameters
In some cases, having function arguments in calldata instead of
memory is more optimal.
    

        EasySign.isWinningSignature (_signature)



Title: Consider inline the following functions to save gas
Severity: GAS


    You can inline the following functions instead of writing a specific function to save gas.
    (see https://github.com/code-423n4/2021-11-nested-findings/issues/167 for a similar issue.)

    
        ConvexStakingWrapper.sol, _getDepositedBalance, { return deposits[_pid][_account].amount; }
        ConvexStakingWrapper.sol, _getTotalSupply, { return IRewardStaking(convexPool[_pid]).balanceOf(address(this)); }



Title: Inline one time use functions
Severity: GAS


The following functions are used exactly once. Therefore you can inline them and save gas and improve code clearness.
    

        ConvexStakingWrapper.sol, _calcRewardIntegral
        ConvexStakingWrapper.sol, _getDepositedBalance
        ConvexStakingWrapper.sol, _getTotalSupply



Title: Check if amount is not zero to save gas
Severity: GAS

The following functions could skip other steps if the amount is 0. (A similar issue: https://github.com/code-423n4/2021-10-badgerdao-findings/issues/82) 

        StakingRewards.sol, recoverERC20



