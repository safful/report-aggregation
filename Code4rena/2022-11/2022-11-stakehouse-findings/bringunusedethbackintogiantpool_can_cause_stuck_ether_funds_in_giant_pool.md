## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-06

# [BringUnusedETHBackIntoGiantPool can cause stuck ether funds in Giant Pool](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/115) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantMevAndFeesPool.sol#L126-L138
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantSavETHVaultPool.sol#L137-L158


# Vulnerability details

## Impact
withdrawUnusedETHToGiantPool will withdraw any eth from the vault if staking has not commenced(knot status is INITIALS_REGISTERED), the eth will be drawn successful to the giant pool. However, idleETH variable is not updated. idleETH  is the available ETH for withdrawing and depositing eth for staking. Since there is no other places that updates idleETH other than depositing eth for staking and withdrawing eth, the eth withdrawn from the vault will be stuck forever. 

## Proof of Concept
place poc in GiantPools.t.sol with `import { MockStakingFundsVault } from "../../contracts/testing/liquid-staking/MockStakingFundsVault.sol";`


```solidity
    function testStuckFundsInGiantMEV() public {

        stakingFundsVault = MockStakingFundsVault(payable(manager.stakingFundsVault()));
        address nodeRunner = accountOne; vm.deal(nodeRunner, 4 ether);
        //address feesAndMevUser = accountTwo; vm.deal(feesAndMevUser, 4 ether);
        //address savETHUser = accountThree; vm.deal(savETHUser, 24 ether);
        address victim = accountFour; vm.deal(victim, 1 ether);


        registerSingleBLSPubKey(nodeRunner, blsPubKeyOne, accountFour);

        emit log_address(address(giantFeesAndMevPool));
        vm.startPrank(victim);

        emit log_uint(victim.balance);
        giantFeesAndMevPool.depositETH{value: 1 ether}(1 ether);
        bytes[][] memory blsKeysForVaults = new bytes[][](1);
        blsKeysForVaults[0] = getBytesArrayFromBytes(blsPubKeyOne);

        uint256[][] memory stakeAmountsForVaults = new uint256[][](1);
        stakeAmountsForVaults[0] = getUint256ArrayFromValues(1 ether);
        giantFeesAndMevPool.batchDepositETHForStaking(getAddressArrayFromValues(address(stakingFundsVault)),getUint256ArrayFromValues(1 ether) , blsKeysForVaults, stakeAmountsForVaults);

        emit log_uint(victim.balance);


        vm.warp(block.timestamp + 60 minutes);
        LPToken lp = (stakingFundsVault.lpTokenForKnot(blsKeysForVaults[0][0]));
        LPToken [][] memory lpToken = new LPToken[][](1);
        LPToken[] memory temp  = new LPToken[](1);
        temp[0] = lp;
        lpToken[0] = temp;

        emit log_uint(address(giantFeesAndMevPool).balance);
        giantFeesAndMevPool.bringUnusedETHBackIntoGiantPool(getAddressArrayFromValues(address(stakingFundsVault)),lpToken, stakeAmountsForVaults);

        emit log_uint(address(giantFeesAndMevPool).balance);
        vm.expectRevert();
        giantFeesAndMevPool.batchDepositETHForStaking(getAddressArrayFromValues(address(stakingFundsVault)),getUint256ArrayFromValues(1 ether) , blsKeysForVaults, stakeAmountsForVaults);

        vm.expectRevert();
        giantSavETHPool.withdrawETH(1 ether);

        vm.stopPrank();
    }


``` 
both withdrawing eth for user and depositing eth to stake fails and reverts as shown in the poc due to underflow in idleETH

Note that the same problem also exists in GiantSavETHVaultPool, however a poc cannot be done for it as another bug exist in GiantSavETHVaultPool which prevents it from receiving funds as it lacks a receive() or fallback() implementation.

## Tools Used

Foundry

## Recommended Mitigation Steps
update `idleETH`  in withdrawUnusedETHToGiantPool