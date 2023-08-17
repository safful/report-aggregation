## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Fund loss or theft by attacker with creating a flash loan and setting SuperVault as receiver so executeOperation()  will be get called by lendingPool but with attackers specified params](https://github.com/code-423n4/2022-04-mimo-findings/issues/123) 

# Lines of code

https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/supervaults/contracts/SuperVault.sol#L76-L99


# Vulnerability details

## Impact
According to Aave documentation, when requesting flash-loan, it's possible to specify a `receiver`, so function `executeOperation()` of that `receiver` will be called by `lendingPool`.
https://docs.aave.com/developers/v/2.0/guides/flash-loans
In the `SuperVault` there is no check to prevent this attack so attacker can use this and perform  `griefing attack` and make miner contract lose all its funds. or he can create specifically crafted `params` so when `executeOperation()` is called by `lendingPool`, attacker could steal vault's user funds.


## Proof of Concept
To exploit this attacker will do this steps:
1. will call `Aave lendingPool` to get a flash-loan and specify `SuperVault` as `receiver` of flash-loan. and also create a specific `params` that invoke `Operation.REBALANCE` action to change user vault's collateral.
2. `lendingPool` will call `executeOperation()` of `SuperVault` with attacker specified data.
3. `executeOperation()` will check `msg.sender` and will process the function call which will cause some dummy exchanges that will cost user exchange fee and flash-loan fee.
4. attacker will repeat this attack until user losses all his funds.
```
  function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address,
    bytes calldata params
  ) external returns (bool) {
    require(msg.sender == address(lendingPool), "SV002");
    (Operation operation, bytes memory operationParams) = abi.decode(params, (Operation, bytes));
    IERC20 asset = IERC20(assets[0]);
    uint256 flashloanRepayAmount = amounts[0] + premiums[0];
    if (operation == Operation.LEVERAGE) {
      leverageOperation(asset, flashloanRepayAmount, operationParams);
    }
    if (operation == Operation.REBALANCE) {
      rebalanceOperation(asset, amounts[0], flashloanRepayAmount, operationParams);
    }
    if (operation == Operation.EMPTY) {
      emptyVaultOperation(asset, amounts[0], flashloanRepayAmount, operationParams);
    }

    asset.approve(address(lendingPool), flashloanRepayAmount);
    return true;
  }
```

To steal user fund in `SupperVault` attacker needs more steps. in all these actions (`Operation.REBALANCE`, `Operation.LEVERAGE`, `Operation.EMPTY`) contract will call `aggregatorSwap()` with data that are controlled by attacker.
```
  function aggregatorSwap(
    uint256 dexIndex,
    IERC20 token,
    uint256 amount,
    bytes memory dexTxData
  ) internal {
    (address proxy, address router) = _dexAP.dexMapping(dexIndex);
    require(proxy != address(0) && router != address(0), "SV201"); 
    token.approve(proxy, amount);
    router.call(dexTxData);
  }
```

Attacker can put special data in `dexTxData` that make contract to do an exchange with bad price. To do this, attacker will create a smart contract that will do this steps:
1. manipulate price in exchange with flash loan.
2. make a call to `executeOperation()` by `Aave flash-loan` with `receiver` and specific `params` so that `SuperVault` will make calls to manipulated exchange for exchanging.
3. do the reverse of #1 and pay the flash-loan and steal the user fund.
The details are:
Attacker can manipulate swapping pool price with flash-loan, then Attacker will create specific `params` and perform steps 1 to 4. so contract will try to exchange tokens and because of attacker price manipulation and specific `dexTxData`, contract will have bad deals.
After that, attacker can reverse the process of swap manipulation and get his  flash-loan tokens and some of `SuperVault` funds and. then pay the flash-loan.

## Tools Used
VIM

## Recommended Mitigation Steps
There should be some state variable which stores the fact that `SuperVault` imitated flash-loan.
When contract tries to start flash-loan, it sets the `isFlash` to `True` and `executeOperation()` only accepts calls if `isFlash` is `True`. and after the flash loan code will set `isFlash` to `False.`

