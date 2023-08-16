## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [add() in the MasterChef.sol contract is marked Public.](https://github.com/code-423n4/2022-02-concur-findings/issues/40) 

# Handle

0x510c


# Vulnerability details

## Impact
The function definition of "add" is marked "public". However, it is never directly called by another function in the same contract or in any of its descendants. Consider to mark it as
"external" instead.

## Proof of Concept

MasterChef.sol

function add(address _token, uint _allocationPoints, uint16 _depositFee, uint _startBlock) public onlyOwner {
        require(_token != address(0), "zero address");
        uint lastRewardBlock = block.number > _startBlock ? block.number : _startBlock;
        totalAllocPoint = totalAllocPoint.add(_allocationPoints);
        require(pid[_token] == 0, "already registered"); // pid starts from 0
        poolInfo.push(
            PoolInfo({
                depositToken: IERC20(_token),
                allocPoint: _allocationPoints,
                lastRewardBlock: lastRewardBlock,
                accConcurPerShare: 0,
                depositFeeBP: _depositFee
            })
        );
        pid[_token] = poolInfo.length - 1;
    }

## Tools Used
Manual Review

## Recommended Mitigation Steps
It recommended to change the visibility of the function to External to optimize the usage of gas.

