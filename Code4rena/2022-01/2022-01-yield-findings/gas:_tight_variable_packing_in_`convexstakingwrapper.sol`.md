## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Tight variable packing in `ConvexStakingWrapper.sol`](https://github.com/code-423n4/2022-01-yield-findings/issues/58) 

# Handle

Dravee


# Vulnerability details

## Impact
Solidity contracts have contiguous 32 bytes (256 bits) slots used in storage. By arranging the variables, it is possible to minimize the number of slots used within a contract's storage and therefore reduce deployment costs.

## Proof of Concept
In `ConvexStakingWrapper.sol`, the order of variables is this way:
```
    uint256 public cvx_reward_integral;
    uint256 public cvx_reward_remaining;
    mapping(address => uint256) public cvx_reward_integral_for;
    mapping(address => uint256) public cvx_claimable_reward;

    //constants/immutables
    address public constant convexBooster = address(0xF403C135812408BFbE8713b5A23a04b3D48AAE31);
    address public constant crv = address(0xD533a949740bb3306d119CC777fa900bA034cd52);
    address public constant cvx = address(0x4e3FBD56CD56c3e72c1403e103b45Db9da5B9D2B);
    address public curveToken;
    address public convexToken;
    address public convexPool;
    address public collateralVault;
    uint256 public convexPoolId;

    //rewards
    RewardType[] public rewards;

    //management
    bool public isShutdown;
    bool private _status;

    bool private constant _NOT_ENTERED = false;
    bool private constant _ENTERED = true;
```
`address` type variables are each of 20 bytes size (way less than 32 bytes). However, they here take up a whole 32 bytes slot (they are contiguous).
As `bool` type variables are of size 1 byte, there's a slot here that can get saved by moving them closer to an address

## Recommended Mitigation Steps
I suggest the following (see the @audit-info tags for more details about what moved and why):
```
    uint256 public cvx_reward_integral;
    uint256 public cvx_reward_remaining;
    mapping(address => uint256) public cvx_reward_integral_for;
    mapping(address => uint256) public cvx_claimable_reward;

    //constants/immutables
    uint256 public convexPoolId; //@audit-info this moved up to free collateralVault's slot.
    address public constant convexBooster = address(0xF403C135812408BFbE8713b5A23a04b3D48AAE31);
    address public constant crv = address(0xD533a949740bb3306d119CC777fa900bA034cd52);
    address public constant cvx = address(0x4e3FBD56CD56c3e72c1403e103b45Db9da5B9D2B);
    address public curveToken;
    address public convexToken;
    address public convexPool;
    address public collateralVault; //@audit-info this got freed from convexPoolId. Slot N is at 20/32 here

    //management 
    bool public isShutdown; //@audit-info this moved up. Slot N is full at 21/32 here 
    bool private _status; //@audit-info this moved up. Slot N is full at 22/32 here

    bool private constant _NOT_ENTERED = false; //@audit-info this moved up but doesn't take a slot as it's constant
    bool private constant _ENTERED = true; //@audit-info this moved up but doesn't take a slot as it's constant
	
    //rewards
    RewardType[] public rewards;
```



