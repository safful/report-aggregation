## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-31

# [Medium: Vaults can be griefed to not be able to be used for deposits](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/422) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/ETHPoolLPFactory.sol#L111


# Vulnerability details

## Description

Interaction with SavETHVault and StakingFundsVault require a minimum amount of MIN_STAKING_AMOUNT. In order to be used for staking, there needs to be 24 ETH or 4 ETH for the desired BLS public key in those vaults. The issue is that vaults can be griefed and made impossible to use for depositing by constantly making sure the *remaining* amount to be added to complete the deposit to the maxStakingAmountPerValidator, is under MIN_STAKING_AMOUNT.

In \_depositETHForStaking:
```
function _depositETHForStaking(bytes calldata _blsPublicKeyOfKnot, uint256 _amount, bool _enableTransferHook) internal {
    require(_amount >= MIN_STAKING_AMOUNT, "Min amount not reached");
    require(_blsPublicKeyOfKnot.length == 48, "Invalid BLS public key");
    // LP token issued for the KNOT
    // will be zero for a new KNOT because the mapping doesn't exist
    LPToken lpToken = lpTokenForKnot[_blsPublicKeyOfKnot];
    if(address(lpToken) != address(0)) {
        // KNOT and it's LP token is already registered
        // mint the respective LP tokens for the user
        // total supply after minting the LP token must not exceed maximum staking amount per validator
        require(lpToken.totalSupply() + _amount <= maxStakingAmountPerValidator, "Amount exceeds the staking limit for the validator");
        // mint LP tokens for the depoistor with 1:1 ratio of LP tokens and ETH supplied
        lpToken.mint(msg.sender, _amount);
        emit LPTokenMinted(_blsPublicKeyOfKnot, address(lpToken), msg.sender, _amount);
    }
    else {
	
        // check that amount doesn't exceed max staking amount per validator
        require(_amount <= maxStakingAmountPerValidator, "Amount exceeds the staking limit for the validator");
    ...    

```


MED - Can grief vaults (SavETHVault, StakingFundsVault) and make them not able to be used for staking by depositing so that left to stake is < MIN_STAKING_AMOUNT. Then it will fail maxStakingAmount check @ _depositEthForStaking

## Impact

Vaults can be griefed to not be able to be used for deposits

## Proof of Concept

1. savETHVault has 22 ETH for some validator
2. Attacker deposits 1.9991 ETH to the savETHVault
3. vault now has 23.9991 ETH. The remaining to complete to 24 is 0.0009 ETH which is under 0.001 ether, min staking amount
4. No one can complete the staking

Note that depositers may try to remove their ETH and redeposit it to complete the deposit to 24. However attack may still keep the delta just under MIN_STAKING_AMOUNT.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Handle the case where the remaining amount to be completed is smaller than MIN_STAKING_AMOUNT, and allow the deposit in that case.