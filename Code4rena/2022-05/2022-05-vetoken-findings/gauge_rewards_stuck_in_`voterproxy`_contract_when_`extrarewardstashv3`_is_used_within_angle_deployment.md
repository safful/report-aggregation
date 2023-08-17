## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Gauge Rewards Stuck In `VoterProxy` Contract When `ExtraRewardStashV3` Is Used Within Angle Deployment](https://github.com/code-423n4/2022-05-vetoken-findings/issues/209) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L495


# Vulnerability details

> Note: This report aims to discuss the issue encountered when `ExtraRewardStashV3` is used within Angle Deployment. There is also another issue when `ExtraRewardStashV2` is used within Angle Deployment, but I will raise it in a separate report since `ExtraRewardStashV2` and `ExtraRewardStashV3` operate differently, and the proof-of-concept and mitigation are different too.

## Proof-of-Concept

In this example, assume the following Angle's gauge setup

> Name = Angle sanDAI_EUR Gauge
>
> Symbol = SsanDAI_EUR
>
> reward_count = 2
>
> reward_tokens(0) = ANGLE
>
> reward_tokens(1) = DAI
>
> Gauge Contract: [LiquidityGaugeV4.vy](https://github.com/AngleProtocol/angle-core/blob/4d854e0d74be703a3707898f26ea2dd4166bc9b6/contracts/staking/LiquidityGaugeV4.vy)
>
> Stash Contract: [ExtraRewardStashV3](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV3.sol)

To collect the gauge rewards, users would trigger the `Booster._earmarkRewards` function to claim veAsset and extra rewards from a gauge. 

Per the code logic, the function will attempt to execute the following two key operations:

1) First Operation -  Claim the veAsset by calling `VoterProxy.claimVeAsset`. Call Flow as follow: `VoterProxy.claimVeAsset() > IGauge(_gauge).claim_rewards()`.
2) Second Operation - Claim extra rewards by calling `ExtraRewardStashV3.claimRewards`. Call flow as follows: `ExtraRewardStashV3.claimRewards > Booster.claimRewards > VoterProxy.claimRewards > IGauge(_gauge).claim_rewards()` . 

Note that`IGauge(_gauge).claim_rewards()` will claim all available reward tokens from the Angle's gauge.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L495](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/Booster.sol#L495)

```solidity
//claim veAsset and extra rewards and disperse to reward contracts
function _earmarkRewards(uint256 _pid) internal {
    PoolInfo storage pool = poolInfo[_pid];
    require(pool.shutdown == false, "pool is closed");

    address gauge = pool.gauge;

    //claim veAsset
    IStaker(staker).claimVeAsset(gauge);

    //check if there are extra rewards
    address stash = pool.stash;
    if (stash != address(0)) {
        //claim extra rewards
        IStash(stash).claimRewards();
        //process extra rewards
        IStash(stash).processStash();
    }
	..SNIP..
}
```

### First Operation -  Claim the veAsset

Since this is a Angle Deployment, when the `VoterProxy.claimVeAsset` is triggered,  it will go through the if-else logic (`escrowModle == IVoteEscrow.EscrowModle.ANGLE`) and execute ` IGauge(_gauge).claim_rewards()`, and all rewards tokens will be sent to `VoterProxy` contract. Assume that `100 ANGLE` and `100 DAI` were received.

Note that in this example, we have two reward tokens (ANGLE and DAI). Additionally, gauge redirection was not configured on the gauge at this point, thus the gauge rewards will be sent to the caller, which is the `VoterProxy` contract.

Subsequently, the code `IERC20(veAsset).safeTransfer(operator, _balance);` will be executed, and veAsset (`100 ANGLE`) reward tokens will be transferred to the `Booster` contract for distribution. However, the `100 DAI` reward tokens will remain stuck in the `VoterProxy` contract. As such, users will not be able to get any reward tokens (e.g. DAI, WETH) except veAsset (ANGLE) tokens from the gauges.

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VoterProxy.sol#L224](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/VoterProxy.sol#L224)

```solidity
function claimVeAsset(address _gauge) external returns (uint256) {
    require(msg.sender == operator, "!auth");

    uint256 _balance = 0;

    if (escrowModle == IVoteEscrow.EscrowModle.PICKLE) {
        try IGauge(_gauge).getReward() {} catch {
            return _balance;
        }
    } else if (
        escrowModle == IVoteEscrow.EscrowModle.CURVE ||
        escrowModle == IVoteEscrow.EscrowModle.RIBBON
    ) {
        try ITokenMinter(minter).mint(_gauge) {} catch {
            return _balance;
        }
    } else if (escrowModle == IVoteEscrow.EscrowModle.IDLE) {
        try ITokenMinter(minter).distribute(_gauge) {} catch {
            return _balance;
        }
    } else if (escrowModle == IVoteEscrow.EscrowModle.ANGLE) {
        try IGauge(_gauge).claim_rewards() {} catch {
            return _balance;
        }
    }

    _balance = IERC20(veAsset).balanceOf(address(this));
    IERC20(veAsset).safeTransfer(operator, _balance);

    return _balance;
}
```

Following is Angle's Gauge Contract for reference:

[https://github.com/AngleProtocol/angle-core/blob/4d854e0d74be703a3707898f26ea2dd4166bc9b6/contracts/staking/LiquidityGaugeV4.vy#L344](https://github.com/AngleProtocol/angle-core/blob/4d854e0d74be703a3707898f26ea2dd4166bc9b6/contracts/staking/LiquidityGaugeV4.vy#L344)

(Mainnet Deployed Address: https://etherscan.io/address/0x8E2c0CbDa6bA7B65dbcA333798A3949B07638026) 

> Note: Angle Protocol is observed to use LiquidityGaugeV4 contract for all of their gauges. Thus, ExtraRewardStashV3 is utilised during pool creation.

```python
@external
@nonreentrant('lock')
def claim_rewards(_addr: address = msg.sender, _receiver: address = ZERO_ADDRESS):
    """
    @notice Claim available reward tokens for `_addr`
    @param _addr Address to claim for
    @param _receiver Address to transfer rewards to - if set to
                     ZERO_ADDRESS, uses the default reward receiver
                     for the caller
    """
    if _receiver != ZERO_ADDRESS:
        assert _addr == msg.sender  # dev: cannot redirect when claiming for another user
    self._checkpoint_rewards(_addr, self.totalSupply, True, _receiver)
```

### Second Operation - Claim extra rewards

After the `IStaker(staker).claimVeAsset(gauge);` code within the `Booster._earmarkRewards` function is executed, `IStash(stash).claimRewards();`  and `IStash(stash).processStash();` functions will be executed next. `stash` == `ExtraRewardStashV3`.

The `ExtraRewardStashV3.claimRewards` will call the `Booster.setGaugeRedirect` first so that all the gauge rewards will be redirected to `ExtraRewardStashV3` stash contract. Subsequently, `ExtraRewardStashV3.claimRewards` will trigger `Booster.claimRewards` to claim the gauge rewards from the Angle's gauge. 

Note that this is the second time the contract attempts to claim gauge rewards from the gauge. Thus, no gauge rewards will be received since we already claimed them earlier. Next, `ExtraRewardStashV3` will attempt to process all the tokens stored in its contract and send them to the respective reward contracts for distribution to the users. However, the contract does not have any tokens stored in it because the earlier attempt to claim gauge rewards return nothing.

As we can see, the DAI reward tokens are still stuck in the `VoterProxy` contract at this point.

[https://github.com/AngleProtocol/angle-core/blob/4d854e0d74be703a3707898f26ea2dd4166bc9b6/contracts/staking/LiquidityGaugeV4.vy#L332](https://github.com/AngleProtocol/angle-core/blob/4d854e0d74be703a3707898f26ea2dd4166bc9b6/contracts/staking/LiquidityGaugeV4.vy#L332)

```python
def set_rewards_receiver(_receiver: address):
    """
    @notice Set the default reward receiver for the caller.
    @dev When set to ZERO_ADDRESS, rewards are sent to the caller
    @param _receiver Receiver address for any rewards claimed via `claim_rewards`
    """
    self.rewards_receiver[msg.sender] = _receiver
```

[https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV3.sol#L61](https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV3.sol#L61)

```solidity
//try claiming if there are reward tokens registered
function claimRewards() external returns (bool) {
    require(msg.sender == operator, "!authorized");

    //this is updateable from v2 gauges now so must check each time.
    checkForNewRewardTokens();

    //make sure we're redirected
    if (!hasRedirected) {
        IDeposit(operator).setGaugeRedirect(pid);
        hasRedirected = true;
    }

    uint256 length = tokenCount;
    if (length > 0) {
        //claim rewards on gauge for staker
        //using reward_receiver so all rewards will be moved to this stash
        IDeposit(operator).claimRewards(pid, gauge);
    }
    return true;
}
```

## Impact

User's gauge rewards are frozen/stuck in `VoterProxy` contract. Additionally, there is no method to sweep/collect the reward tokens stuck in the `VoterProxy` contract.

## Recommended Mitigation Steps

> Note: I do not see `Booster.setGaugeRedirect` being called in the deployment and testing scripts. Thus, it is fair to assume that the team is not aware of the need to trigger `Booster.setGaugeRedirect` during deployment. If the gauge redirection has been set to the stash contract `ExtraRewardStashV3` right from the start before anyone triggered the `earmarkRewards` function, this issue should not occur.

Consider triggering `Booster.setGaugeRedirect` during the deployment to set gauge redirection to stash contract (`ExtraRewardStashV3`) so that the Angle's gauge rewards will not be redirected to `VoterProxy` contract and get stuck there.

Alternatively, update the `Booster._earmarkRewards` to as follows:

```solidity
//claim veAsset and extra rewards and disperse to reward contracts
function _earmarkRewards(uint256 _pid) internal {
	PoolInfo storage pool = poolInfo[_pid];
	require(pool.shutdown == false, "pool is closed");

	address stash = pool.stash;
	if (escrowModle == IVoteEscrow.EscrowModle.ANGLE) {
		//claims gauges rewards
		IStash(stash).claimRewards();
		//process gauges rewards
		IStash(stash).processStash();
	} else {
		//claim veAsset
        IStaker(staker).claimVeAsset(gauge);

        //check if there are extra rewards
        address stash = pool.stash;
        if (stash != address(0)) {
            //claim extra rewards
            IStash(stash).claimRewards();
            //process extra rewards
            IStash(stash).processStash();
        }
	}

	//veAsset balance
    uint256 veAssetBal = IERC20(veAsset).balanceOf(address(this));
	..SNIP..
}
```

There is no need to specifically call `VoterProxy.claimVeAsset` to fetch ANGLE for Angle Protocol because calling `IStash(stash).claimRewards()` will fetch both ANGLE and other reward tokens from the gauge anyway. When the stash contract receives the ANGLE tokens, it will automatically transfer all of them back to `Booster` contract when `IStash(stash).processStash()` is executed. The `IStash(stash).claimRewards()` function also performs a sanity check to ensure that the gauge redirection is pointing to itself before claiming the gauge rewards, and automatically configure them if it is not, so it will not cause the reward tokens to get stuck in `VoterProxy` contract.

- Curve uses an older version of LiquidityGauge contract. Thus, two calls are needed (`Minter.mint` to claim CRV and `LiquidityGauge.claim_rewards` to claim other rewards). 

- Angle uses newer version of LiquidityGauge (V4) contract that just need one function call (`LiquidityGauge.claim_rewards` ) to fetch both veAsset and other rewards.
- IDLE uses LiquidityGauge (V3) contract. veAsset (IDLE) is minted by calling `DistributorProxy.distribute` and gauge rewards are claimed by calling `LiquidityGauge.claim_rewards`.

Due to the discrepancies between different protocols in the reward claiming process, additional care must be taken to ensure that the flow of veAsset and gauge rewards are transferred to the appropriate contracts during integration. Otherwise, rewards will be stuck.

Lastly, I only see test cases written for claiming veAsset from the gauge. For completeness, it is recommended to also write test cases for claiming extra rewards from the gauge apart from veAsset.

