## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-concur-findings/issues/263) 

# QA Report
**Table of Contents:**

- [QA Report](#qa-report)
  - [Transfers](#transfers)
    - [Prevent accidentally burning tokens](#prevent-accidentally-burning-tokens)
    - [Use safeTransfer or require()/conditional instead of transfer/transferFrom](#use-safetransfer-or-requireconditional-instead-of-transfertransferfrom)
    - [Use SafeERC20.safeApprove()](#use-safeerc20safeapprove)
  - [Libraries](#libraries)
    - [Deprecated library used for Solidity 0.8.11: SafeMath](#deprecated-library-used-for-solidity-0811-safemath)
  - [Variables](#variables)
    - [Missing Address(0) checks](#missing-address0-checks)
    - [Variables that should be constant](#variables-that-should-be-constant)
    - [Variables that are assumed to be initialized before a function call, but might not be](#variables-that-are-assumed-to-be-initialized-before-a-function-call-but-might-not-be)
    - [Variables that should be bounded](#variables-that-should-be-bounded)
    - [Variables that should be grouped together in a struct](#variables-that-should-be-grouped-together-in-a-struct)
      - [File: ConvexStakingWrapper.sol](#file-convexstakingwrappersol)
      - [File: Shelter.sol](#file-sheltersol)
  - [Functions](#functions)
    - [Functions that should be declared external](#functions-that-should-be-declared-external)
  - [Arithmetics](#arithmetics)
    - [Possible division by 0](#possible-division-by-0)

## Transfers
### Prevent accidentally burning tokens

Transferring tokens to the zero address is usually prohibited to accidentally avoid "burning" tokens by sending them to an unrecoverable zero address.

Places where I couldn't find a zero address check (or where the destination isn't a zero-checked address):
```
ConvexStakingWrapper.sol:179:            IERC20(reward.token).transfer(treasury, d_reward / 5); //@audit treasury isn't address(0) checked
ConvexStakingWrapper.sol:182:        IERC20(reward.token).transfer(address(claimContract), d_reward);//@audit claimContract isn't address(0) checked
MasterChef.sol:206:            transferSuccess = concur.transfer(_to, concurBalance);//@audit _to == _recipient and isn't address(0) checked
MasterChef.sol:208:            transferSuccess = concur.transfer(_to, _amount); //@audit _to == _recipient and isn't address(0) checked
``` 

I suggest adding a check to prevent accidentally burning tokens.

### Use safeTransfer or require()/conditional instead of transfer/transferFrom

Silent failures (lack of failure detection / revert in case of failure) may happen here:
```
File: ConvexStakingWrapper.sol
178:         if (reward.token == cvx || reward.token == crv) {
179:             IERC20(reward.token).transfer(treasury, d_reward / 5); //@audit return value ignored
180:             d_reward = (d_reward * 4) / 5;
181:         }
182:         IERC20(reward.token).transfer(address(claimContract), d_reward);//@audit return value ignored
```
Consider using safeTransfer. That's already the case at other places on the same contract

### Use SafeERC20.safeApprove()
`approve()` will fail for certain token implementations that do not return a boolean value. It is recommended to use OpenZeppelin's SafeERC20's safeApprove().

Instances include:
```
USDMPegRecovery.sol:79:        usdm.approve(address(usdm3crv), addingLiquidity);
USDMPegRecovery.sol:80:        pool3.approve(address(usdm3crv), addingLiquidity);
```

## Libraries

### Deprecated library used for Solidity 0.8.11: SafeMath
Use Solidity 0.8.*'s default checks instead:

```
MasterChef.sol:10:import "@openzeppelin/contracts/utils/math/SafeMath.sol";
MasterChef.sol:14:    using SafeMath for uint;
```

## Variables

### Missing Address(0) checks
```
		- rewardNotifier = _notifier (contracts/ConcurRewardPool.sol#16)
		- treasury = _treasury (contracts/ConvexStakingWrapper.sol#70)
		- treasury = _treasury (contracts/ConvexStakingWrapper.sol#83)
		- rewardsDistribution = _rewardsDistribution (contracts/StakingRewards.sol#45)
		- rewardsDistribution = _rewardsDistribution (contracts/StakingRewards.sol#195)
		- kpiOracle = _kpiOracle (contracts/USDMPegRecovery.sol#57)
		- (success,result) = _to.call{value: _value}(_data) (contracts/VoteProxy.sol#33)
```

### Variables that should be constant
```
MasterChef._concurShareMultiplier (contracts/MasterChef.sol#56)
MasterChef._perMille (contracts/MasterChef.sol#57)
MasterChef.concurPerBlock (contracts/MasterChef.sol#50)
```

### Variables that are assumed to be initialized before a function call, but might not be
```
File: ConvexStakingWrapper.sol
50:     IConcurRewardClaim public claimContract;
...
86:     function setRewardPool(address _claimContract) external onlyOwner {
87:         claimContract = IConcurRewardClaim(_claimContract);
88:     }
```
### Variables that should be bounded

The variable `MasterChef.sol:43: uint16 depositFeeBP;  // Deposit fee in basis points` is never bounded, and UInt16.MaxValue is 65535

### Variables that should be grouped together in a struct

For maps that use the same key value: having separate fields is error prone (like in case of deletion or future new fields).

#### File: ConvexStakingWrapper.sol

6 maps can be grouped together, as they use the same `pid`:
```
41:     //convex rewards
42:     mapping(uint256 => address) public convexPool;
43:     mapping(uint256 => RewardType[]) public rewards;
44:     mapping(uint256 => mapping(uint256 => mapping(address => Reward)))
45:         public userReward;
46:     mapping(uint256 => mapping(address => uint256)) public registeredRewards;
...
63:     mapping(uint256 => mapping(address => Deposit)) public deposits;
64:     mapping(uint256 => mapping(address => WithdrawRequest)) public withdrawRequest;
```
I'd suggest these 3 related data get grouped in a struct, let's name it `RewardInfo`:  
```
struct RewardInfo {  
  address convexPool;  
  RewardType[] rewards;  
  mapping(uint256 => mapping(address => Reward)) userReward;  
  mapping(address => uint256) registeredRewards;  
  mapping(address => Deposit) deposits;  
  mapping(address => WithdrawRequest) withdrawRequest;  
}  
```
And it would be used as a state variable in this manner (where `uint256` is `pid`):  
```  
mapping(uint256 => RewardInfo) rewardInfo;  
```  

#### File: Shelter.sol
3 maps can be grouped together, as they use the same `_token`:
```  
17:     mapping(IERC20 => mapping(address => bool)) public override claimed;
18: 
19:     mapping(IERC20 => uint256) public activated;
20: 
21:     mapping(IERC20 => uint256) public savedTokens;
```
I'd suggest these 3 related data get grouped in a struct, let's name it `TokenInfo`:  
```
struct TokenInfo {  
  mapping(address => bool) claimed;  
  uint256 activated;  
  uint256 savedTokens;  
}  
```
And it would be used as a state variable in this manner (where `IERC20` is `_token`):  
```  
mapping(IERC20 => TokenInfo) tokenInfo;  
```  

## Functions

### Functions that should be declared external
```
	- ConvexStakingWrapper.addRewards(uint256) (contracts/ConvexStakingWrapper.sol#93-140)
	- MasterChef.add(address,uint256,uint16,uint256) (contracts/MasterChef.sol#86-101)
	- MasterChef.massUpdatePools() (contracts/MasterChef.sol#127-132)
```

## Arithmetics

### Possible division by 0
There are no checks that the denominator is `!= 0` at thoses lines:
```
library\CvxMining.sol:16:        uint256 cliff = supply / reductionPerCliff;
MasterChef.sol:120:            uint concurReward = multiplier.mul(concurPerBlock).mul(pool.allocPoint).div(totalAllocPoint);
MasterChef.sol:121:            accConcurPerShare = accConcurPerShare.add(concurReward.mul(_concurShareMultiplier).div(lpSupply));
MasterChef.sol:151:        uint concurReward = multiplier.mul(concurPerBlock).mul(pool.allocPoint).div(totalAllocPoint);
MasterChef.sol:152:        pool.accConcurPerShare = pool.accConcurPerShare.add(concurReward.mul(_concurShareMultiplier).div(lpSupply));
Shelter.sol:54:        uint256 amount = savedTokens[_token] * client.shareOf(_token, msg.sender) / client.totalShare(_token);
```