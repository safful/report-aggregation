## Tags

- bug
- 2 (Med Risk)
- high quality report
- resolved
- sponsor confirmed
- selected for report

# [It is possible that receiver and treasury can receive nothing when calling `withdraw` function due to division being performed before multiplication](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/378) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Vault.sol#L378-L426
https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Vault.sol#L203-L234


# Vulnerability details

## Impact
In the following `beforeWithdraw` function, `entitledAmount = amount.divWadDown(idFinalTVL[id]).mulDivDown(idClaimTVL[id], 1 ether)` can be executed in several places. Because it uses division before multiplication, it is possible that `entitledAmount` is calculated to be 0. As the `withdraw` function shows below, when `entitledAmount` is 0, the receiver and treasury both receive 0. As a result, calling `withdraw` with a positive `assets` input can still result in transferring nothing to the receiver and treasury.

https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Vault.sol#L378-L426
```solidity
    function beforeWithdraw(uint256 id, uint256 amount)
        public
        view
        returns (uint256 entitledAmount)
    {
        // in case the risk wins aka no depeg event
        // risk users can withdraw the hedge (that is paid by the hedge buyers) and risk; withdraw = (risk + hedge)
        // hedge pay for each hedge seller = ( risk / tvl before the hedge payouts ) * tvl in hedge pool
        // in case there is a depeg event, the risk users can only withdraw the hedge
        if (
            keccak256(abi.encodePacked(symbol)) ==
            keccak256(abi.encodePacked("rY2K"))
        ) {
            if (!idDepegged[id]) {
                //depeg event did not happen
                /*
                entitledAmount =
                    (amount / idFinalTVL[id]) *
                    idClaimTVL[id] +
                    amount;
                */
                entitledAmount =
                    amount.divWadDown(idFinalTVL[id]).mulDivDown(
                        idClaimTVL[id],
                        1 ether
                    ) +
                    amount;
            } else {
                //depeg event did happen
                entitledAmount = amount.divWadDown(idFinalTVL[id]).mulDivDown(
                    idClaimTVL[id],
                    1 ether
                );
            }
        }
        // in case the hedge wins aka depegging
        // hedge users pay the hedge to risk users anyway,
        // hedge guy can withdraw risk (that is transfered from the risk pool),
        // withdraw = % tvl that hedge buyer owns
        // otherwise hedge users cannot withdraw any Eth
        else {
            entitledAmount = amount.divWadDown(idFinalTVL[id]).mulDivDown(
                idClaimTVL[id],
                1 ether
            );
        }

        return entitledAmount;
    }
```

https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Vault.sol#L203-L234
```solidity
    function withdraw(
        uint256 id,
        uint256 assets,
        address receiver,
        address owner
    )
        external
        override
        epochHasEnded(id)
        marketExists(id)
        returns (uint256 shares)
    {
        if(
            msg.sender != owner &&
            isApprovedForAll(owner, receiver) == false)
            revert OwnerDidNotAuthorize(msg.sender, owner);

        shares = previewWithdraw(id, assets); // No need to check for rounding error, previewWithdraw rounds up.

        uint256 entitledShares = beforeWithdraw(id, shares);
        _burn(owner, id, shares);

        //Taking fee from the amount
        uint256 feeValue = calculateWithdrawalFeeValue(entitledShares, id);
        entitledShares = entitledShares - feeValue;
        asset.transfer(treasury, feeValue);

        emit Withdraw(msg.sender, receiver, owner, id, assets, entitledShares);
        asset.transfer(receiver, entitledShares);

        return entitledShares;
    }
```

## Proof of Concept
Please append the following test in `test\AssertTest.t.sol`. This test will pass to demonstrate the described scenario.

```solidity
    function testReceiveZeroDueToDivBeingPerformedBeforeMul() public {
        vm.deal(alice, 1e24);
        vm.deal(chad, 1e24);

        vm.startPrank(admin);
        FakeOracle fakeOracle = new FakeOracle(oracleFRAX, STRIKE_PRICE_FAKE_ORACLE);
        vaultFactory.createNewMarket(FEE, tokenFRAX, DEPEG_AAA, beginEpoch, endEpoch, address(fakeOracle), "y2kFRAX_99*");
        vm.stopPrank();

        address hedge = vaultFactory.getVaults(1)[0];
        address risk = vaultFactory.getVaults(1)[1];
        
        Vault vHedge = Vault(hedge);
        Vault vRisk = Vault(risk);

        // alice deposits 1e24 in hedge vault
        vm.startPrank(alice);
        ERC20(WETH).approve(hedge, 1e24);
        vHedge.depositETH{value: 1e24}(endEpoch, alice);
        vm.stopPrank();

        // chad deposits 1e24 in risk vault
        vm.startPrank(chad);
        ERC20(WETH).approve(risk, 1e24);
        vRisk.depositETH{value: 1e24}(endEpoch, chad);
        vm.stopPrank();

        vm.warp(beginEpoch + 10 days);

        // depeg occurs
        controller.triggerDepeg(SINGLE_MARKET_INDEX, endEpoch);

        vm.startPrank(chad);

        // chad withdraws 1e5 from risk vault
        vRisk.withdraw(endEpoch, 1e5, chad, chad);

        // the amount to chad is 0 because division is performed before multiplication
        uint256 entitledShares = vRisk.beforeWithdraw(endEpoch, 1e5);

        // chad receives nothing
        assertEq(entitledShares, 0);
        assertEq(ERC20(WETH).balanceOf(chad), 0);

        // the amount to chad would be positive when multiplication is performed before division
        uint256 entitledShares2 = (1e5 * vRisk.idClaimTVL(endEpoch)) / vRisk.idFinalTVL(endEpoch);
        assertTrue(entitledShares2 > entitledShares);

        vm.stopPrank();
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
`entitledAmount = amount.divWadDown(idFinalTVL[id]).mulDivDown(idClaimTVL[id], 1 ether)` in the `beforeWithdraw` function can be updated to the following code.
```solidity
    entitledAmount = (amount * idClaimTVL[id]) / idFinalTVL[id]
```