## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [```withdrawForETH``` could be used to drain the WETH in ```RubiconRouter.sol```](https://github.com/code-423n4/2022-05-rubicon-findings/issues/356) 

# Lines of code

[RubiconRouter.sol#L475-L492](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L475-L492)


# Vulnerability details

## Impact

In the ```withdrawForETH``` function in ```RubiconRouter.sol```, the ```targetPool``` may be any contract that implements the ```IBathToken``` interface and returns ```wethAddress``` as its underlying token. The ```withdrawnWETH``` amount could be set to the ```RubiconRouter.sol``` contract's WETH balance so that the contract's entire WETH balance is withdrawn, as long as the ```tagetPool``` does not transfer any WETH to ```RubiconRouter.sol```. The caller of the ```withdrawForETH``` function would then receive the withdraw amount.

## Proof of Concept

```
    function withdrawForETH(uint256 shares, address targetPool)
        external
        payable
        returns (uint256 withdrawnWETH)
    {
        IERC20 target = IBathToken(targetPool).underlyingToken();
        require(target == ERC20(wethAddress), "target pool not weth pool");
        require(
            IBathToken(targetPool).balanceOf(msg.sender) >= shares,
            "don't own enough shares"
        );
        IBathToken(targetPool).transferFrom(msg.sender, address(this), shares);
        withdrawnWETH = IBathToken(targetPool).withdraw(shares);
        WETH9(wethAddress).withdraw(withdrawnWETH);

        //Send back withdrawn native eth to sender
        msg.sender.transfer(withdrawnWETH);
    }
```

1. Let ```shares``` be equal to the contracts WETH balance.

2. The malicious ```targetPool``` contract returns the ```wethAddress``` as the underlying token on [line 480](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L480). 

3. ```targetPool``` returns the max uint256 value for its balanceOf function to pass the require condition on [line 483](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L483) for any value of shares.

4. The transferFrom on [line 486](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L486) does not have to do anything and its withdraw function should return the WETH balance of ```RubiconRouter.sol```.

5. The ```RubiconRouter.sol``` contract will then [withdraw ETH](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L488) equal to the ```withdrawWETH``` amount, which should be equal to the contract's WETH balance.

6. The caller of the ```withdrawForETH``` function receives the withdraw ETH without providing any WETH.


## Recommended Mitigation Steps:

Check the contract's WETH balance before the caller is supposed to send the WETH and after the WETH is sent to confirm the contract has received enough WETH from the caller.

