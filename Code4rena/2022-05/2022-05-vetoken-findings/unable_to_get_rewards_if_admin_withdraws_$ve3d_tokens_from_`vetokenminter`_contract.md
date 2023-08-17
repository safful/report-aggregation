## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unable To Get Rewards If Admin Withdraws $VE3D tokens From `VeTokenMinter` Contract](https://github.com/code-423n4/2022-05-vetoken-findings/issues/202) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L48


# Vulnerability details

## Vulernability Details

It was observed that users will not be able to get their rewards from the reward contract at certain point of time if admin withdraws $VE3D token from the `VeTokenMinter` contract.

## Proof-of-Concept

Based on the deployment script, it was understood that at the start of the project deployment, 30 million $VE3D tokens will be pre-minted for the `VeTokenMinter` contract. Thus, the `veToken.balanceOf(VeTokenMinter.address)` will be 30 million $VE3D tokens after the deployment.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/migrations/2_deploy_basic_contracts.js#L18](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/migrations/2_deploy_basic_contracts.js#L18)

```javascript
// vetoken minter
await deployer.deploy(VeTokenMinter, veTokenAddress);
let vetokenMinter = await VeTokenMinter.deployed();
addContract("system", "vetokenMinter", vetokenMinter.address);
global.created = true;
//mint vetoke to minter contract
const vetoken = await VeToken.at(veTokenAddress);
await vetoken.mint(vetokenMinter.address, web3.utils.toWei("30000000"), { from: vetokenOperator });
addContract("system", "vetoken", veTokenAddress);
```

In the `VeTokenMinter ` contract, there is a function called `VeTokenMinter.withdraw` that allows the admin to withdraw $VE3D tokens from the contract. Noted that this withdraw function only perform the transfer, but did not update any of the state variables (e.g. totalSupply, maxSupply) in the contract.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L77](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L77)

```solidity
function withdraw(address _destination, uint256 _amount) external onlyOwner {
    veToken.safeTransfer(_destination, _amount);

    emit Withdraw(_destination, _amount);
}
```

Assuming that an admin withdrawed 29 million $VE3D tokens from the `VoteProxy` with the appropriate approval from the DAO or community for some valid purposes. The `veToken.balanceOf(VeTokenMinter.address)` will be 1 million $VE3D tokens after the withdrawal.

At this point, notice that `veToken.balanceOf(VeTokenMinter.address)` is 1 million, while the `VeTokenMinter.maxSupply` constant is 30 million. Therefore, there exists a discrepency between the actual amount of $VE3D tokens (1 million) stored in the contact versus the max supply (30 million).

This discrepency will cause an issue in the `VeTokenMinter.mint` function because the calculation of the amount of $VE3D tokens to be transferred is based on the fact that 30 million $VE3D tokens is always sitting in the `VeTokenMinter` contract, and thus there is always sufficient $VE3D tokens available in the `VeTokenMinter` contract to send to its users.

The `uint256 amtTillMax = maxSupply.sub(supply);` code shows that the calculation is based on `maxSupply` constant, which is 30 million.

Assume that `mint(0x001, 10 million)` is called, and the value of the state variables when stepping through this function are as follows:

- `maxSupply` constant = 30 million
- `veToken.balanceOf(VeTokenMinter.address)` = 1 million
- `supply` & `totalSupply` = 20 million
- `totalCliffs` = 1000
- `reductionPerCliff ` = 30,000 (maxSupply / totalCliffs)
- `cliff` = 666 (supply/reductionPerCliff)
- `reduction` = 1000 - 666 = 334
- `_amount` = 10 million * (334/1000) = 3.340 million
- `amtTillMax` = 10 million (maxSupply - supply) (Over here the contract assume that it still has 10 million VE3D tokens more to reach the max supply)
- `(_amount > amtTillMax)` = `False` (since "3.340 million > 10 million" = false )
- `veToken.safeTransfer(0x001, 3.340 million)` (This will revert. Insufficent balance)

The `veToken.safeTransfer(0x001, 3.340 million` will fail and revert because `VeTokenMinter` contract does not hold sufficent amount of $VE3D tokens to transfer out.`veToken.balanceOf(VeTokenMinter.address)` = 1 million, while the contract was attempting to send out 3.340 million.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L48](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L48)

```solidity
function mint(address _to, uint256 _amount) external {
    require(operators.contains(_msgSender()), "not an operator");

    uint256 supply = totalSupply;

    //use current supply to gauge cliff
    //this will cause a bit of overflow into the next cliff range
    //but should be within reasonable levels.
    //requires a max supply check though
    uint256 cliff = supply.div(reductionPerCliff);
    //mint if below total cliffs
    if (cliff < totalCliffs) {
        //for reduction% take inverse of current cliff
        uint256 reduction = totalCliffs.sub(cliff);
        //reduce
        _amount = _amount.mul(reduction).div(totalCliffs);

        //supply cap check
        uint256 amtTillMax = maxSupply.sub(supply);
        if (_amount > amtTillMax) {
            _amount = amtTillMax;
        }

        //mint
        veToken.safeTransfer(_to, _amount);
        totalSupply += _amount;
    }
}
```

The failure/revert of `VeTokenMinter.mint` function will cascade up to `Booster.rewardClaimed`, and futher cascade up to `BaseRewardPool.getReward`. Thus, `BaseRewardPool.getReward` will stop working. As a result, the users will not be able to get any rewards from the reward contracts. 

This issue will affect all projects (Curve, Pickle, Ribbon, Idle, Angle, Balancer) because `VeTokenMinter ` contract is deployed once, and referenced by all the projects. Thus, the impact could be quite widespread if this occurs, and many users would be affected.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L598](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L598)

```solidity
function rewardClaimed(
    uint256 _pid,
    address _address,
    uint256 _amount
) external returns (bool) {
    address rewardContract = poolInfo[_pid].veAssetRewards;
    require(msg.sender == rewardContract || msg.sender == lockRewards, "!auth");
    ITokenMinter veTokenMinter = ITokenMinter(minter);
    //calc the amount of veAssetEarned
    uint256 _veAssetEarned = _amount.mul(veTokenMinter.veAssetWeights(address(this))).div(
        veTokenMinter.totalWeight()
    );
    //mint reward tokens
    ITokenMinter(minter).mint(_address, _veAssetEarned);

    return true;
}
```

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/BaseRewardPool.sol#L267](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/BaseRewardPool.sol#L267)

```solidity
function getReward(address _account, bool _claimExtras)
    public
    updateReward(_account)
    returns (bool)
{
    uint256 reward = earned(_account);
    if (reward > 0) {
        rewards[_account] = 0;
        rewardToken.safeTransfer(_account, reward);
        IDeposit(operator).rewardClaimed(pid, _account, reward);
        emit RewardPaid(_account, reward);
    }

    //also get rewards from linked rewards
    if (_claimExtras) {
        for (uint256 i = 0; i < extraRewards.length; i++) {
            IRewards(extraRewards[i]).getReward(_account);
        }
    }
    return true;
}
```

## Recommended Mitigation Steps

Remove the `VeTokenMinter.withdraw` function if possible. Otherwise, update the internal accounting of `VeTokenMinter` contract during withdrawal so that the actual balance of the $VE3D tokens is taken into consideration within the `VeTokenMinter.mint`, and the contract will not attempt to transfer more tokens than what it has.

On a side note, [Convex's Minter contract](https://github.com/convex-eth/platform/blob/main/contracts/contracts/Cvx.sol), will mint the `CRX` gov tokens to the users on the fly. See https://github.com/convex-eth/platform/blob/1f11027d429e454dacc4c959502687eaeffdb74a/contracts/contracts/Cvx.sol#L76. Thus, there will not be a case where there is not sufficient `CRV` tokens in the contract to send to it users.

However, in VeToken Protocol, it attempts to transfer the portion of pre-minted $VE3D tokens (30 millions) to the users. See https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VeTokenMinter.sol#L72. Thus, it is possible that there is not enough $VE3D tokens to send to its users if the admin withdraw the pre-minted $VE3D tokens.

