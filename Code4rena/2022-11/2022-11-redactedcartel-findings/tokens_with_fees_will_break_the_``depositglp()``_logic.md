## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [Tokens with fees will break the ``depositGlp()`` logic](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/196) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGlp.sol#L367
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L583


# Vulnerability details

## Impact
In ``PirexGmx`` and ``AutoPxGlp`` you have function ``depositGlp()``, which accepts any ERC20 token from whitelist
Now there are 9 tokens (see here: https://arbiscan.io/address/0x489ee077994b6658eafa855c308275ead8097c4a#readContract):
``WBTC, WETH, USDC, LINK, UNI, USDT, MIM, FRAX, DAI``
And the list may extend

So any user can deposit any of those tokens and receive pxGlp token:
```
    function testUSDTDepositGlp() external {
        // 0 USDT TOKENS
        address myAddress = address(this);
        assertEq(usdt.balanceOf(myAddress), 0);

        // The one with many USDT tokens
        vm.prank(0xB6CfcF89a7B22988bfC96632aC2A9D6daB60d641);
        uint256 amount = 100000;
        usdt.transfer(myAddress, amount);

        // amount USDT TOKENS
        assertEq(usdt.balanceOf(myAddress), amount);

        // DEPOSIT USDT TOKENS
        usdt.approve(address(pirexGmx), amount);
        pirexGmx.depositGlp(address(usdt), amount, 1, 1, address(this));
        
        // SUCCESSSFULLY DEPOSITED
        assertEq(usdt.balanceOf(address(this)), 0);
        assertEq(pxGlp.balanceOf(address(this)), 118890025839780442);
    }
```

But if of this tokens will start charge fee on transfers, the logic will be broken and calls to ``depositGlp()`` with suck token will fail

Because here you use the amount of tokens sent from user wallet: https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexGmx.sol#L512
```
            t.safeTransferFrom(msg.sender, address(this), tokenAmount);

            // Mint and stake GLP using ERC20 tokens
            deposited = gmxRewardRouterV2.mintAndStakeGlp(
                token,
                tokenAmount,
                minUsdg,
                minGlp
            );
```
And then ``gmxRewardRouterV2`` tries to transfer tokens to his balance from your balance: 
```
IERC20(_token).safeTransferFrom(_fundingAccount, address(vault), _amount);
``` 
(See https://arbiscan.io/address/0x321f653eed006ad1c29d174e17d96351bde22649#code - GlpManager and
https://arbiscan.io/address/0xA906F338CB21815cBc4Bc87ace9e68c87eF8d8F1#code - RewardRouterV2)


But you received less than ``tokenAmount`` tokens because of fee. And transaction will fail


## Proof of Concept
Let's imagine USDT in arbitrub started to charge fees 1% per transfer

Alice wants to deposit 100 USDT through ``PirexGmx.depositGlp()``
Then you do 
``t.safeTransferFrom(Alice, address(this), 100);``
You will receive only 99 USDT

But in the next line you will try to send 100 USDT:
```
deposited = gmxRewardRouterV2.mintAndStakeGlp(
                token,
                tokenAmount,
                minUsdg,
                minGlp
            );
```

So transaction fails and Alice can't get pxGlp tokens 


## Tools Used

vs code

## Recommended Mitigation Steps

USDT already has fees in other blockchains. 
Many of these tokens use proxy pattern (and USDT too). It's quite probably that in one day one of the tokens will start charge fees. Or you would like to add one more token to whitelist and the token will be with fees

Thats why I consider finding as medium severity 

To avoid problems, use common pattern, when you check your balance before operation and balance after, like that:

```
            uint256 balanceBefore = t.balanceOf(address(this));
            t.safeTransferFrom(msg.sender, address(this), tokenAmount);
            uint256 balanceAfter = t.balanceOf(address(this));

            uint256 tokenAmount = balanceAfter - balanceBefore;

            // Mint and stake GLP using ERC20 tokens
            deposited = gmxRewardRouterV2.mintAndStakeGlp(
                token,
                tokenAmount,
                minUsdg,
                minGlp
            );
```
