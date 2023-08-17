## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-13

# [possible reentrancy and fund theft in withdrawDETH() of GiantSavETHVaultPool because there is no whitelist check for user provided Vaults and there is no reentrancy defnece](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/226) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L62-L102


# Vulnerability details

## Impact
Function `withdrawDETH()` in `GiantSavETHVaultPool` allows a user to burn their giant LP in exchange for dETH that is ready to withdraw from a set of savETH vaults. This function make external calls to user provided addresses without checking those addresses and send increased dETH balance of contract during the call to user. user can provide malicious addresses to contract and then took the execution flow during the transaction and increase dETH balance of contract by other calls and make contract to transfer them to him.

## Proof of Concept
This is `withdrawDETH()` in `GiantSavETHVaultPool`  code:
```
    /// @notice Allow a user to burn their giant LP in exchange for dETH that is ready to withdraw from a set of savETH vaults
    /// @param _savETHVaults List of savETH vaults being interacted with
    /// @param _lpTokens List of savETH vault LP being burnt from the giant pool in exchange for dETH
    /// @param _amounts Amounts of giant LP the user owns which is burnt 1:1 with savETH vault LP and in turn that will give a share of dETH
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
As you can see first contract save the dETH balance of contract by this line: `uint256 dETHReceivedFromAllSavETHVaults = getDETH().balanceOf(address(this));` and then it loops through user provided vaults addresses and call those vaults to withdraw dETH and in the end it calculates `dETHReceivedFromAllSavETHVaults` and transfer those dETH to user: ` getDETH().transfer(msg.sender, dETHReceivedFromAllSavETHVaults);`. attacker can perform these steps:
1- create a malicious contract `AttackerVault` which is copy of `SavETHVault` with modifiction.
2- call `withdrawDETH()` with Vault list `[ValidVault1, ValidVault2, AttackerVault, ValidVaul3]`.
3- contract would save the dETH balance of itself and then loops through Vaults to validate and burn LPTokens.
4- contract would reach Vault `AttackerVault` and call attacker controlled address.
5- attacker contract call other functions to increase dETH balance of contract (if it's not possible to increase dETH balance of contract by other way so there is no need to save contract initial balance of dETH before the loop and dETH balance of contract would be zero always)
6- `withdrawDETH()` would finish the loop and transfer all the increase dETH balance to attacker which includes extra amounts.

because contract don't check the provided addresses and calls them and there is no reentrancy defense mechanism there is possibility of reentrancy attack which can cause fund lose.

## Tools Used
VIM

## Recommended Mitigation Steps
check the provided addresses and also have some reentrancy defence mechanisim.