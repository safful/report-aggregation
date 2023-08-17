## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Notional

# [DOS set token through erc777 hook](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/168) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/main/index-coop-notional-trade-module/contracts/protocol/modules/v1/DebtIssuanceModule.sol#L131-L141
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC777/ERC777.sol#L376-L380


# Vulnerability details

## Impact
The `wfCash` is an `erc777` token. [ERC777.sol#L376-L380](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC777/ERC777.sol#L376-L380) Users can get the control flow before sending token and after receiving tokens. This creates attack vectors that require extra caution in designing modules. Any combination of modules may lead to a possible exploit. To elaborate on the dangerousness of the re-entrancy attack, a possible scenario is presented.

Before the exploit, we first elaborate on three attack vectors:

1. [DebtIssuanceModule.sol#L131-L141](https://github.com/code-423n4/2022-06-notional-coop/blob/main/index-coop-notional-trade-module/contracts/protocol/modules/v1/DebtIssuanceModule.sol#L131-L141) The issuance module would pull tokens from the sender before minting setToken.
Assume there are three compoenents in this set. 1. CDai. 2. wfCash  In the `_callTokensToSend`, the setToken has received `cdai` and the `totalSupply` is still the same.

2. `nonReentrant` does not protect cross-contract re-entrancy. This means, that during the `issue` of issuance module, users can trigger other modules' functions.

3. Restricted functions with `onlyManagerAndValidSet` modifier may be triggered by the exploiter as well. Manager of a setToken is usually a manager contract. Assume it's a multisig-wallet, the exploiter can front-run the execute transaction and replay the payload during his exploit. Note, a private transaction from flash-bot can still be front-run. Please refer to the [uncle bandit risk](https://docs.flashbots.net/flashbots-protect/rpc/uncle-bandits)


Given the above attack vectors, the exploiter have enough weapons to exploit the `setToken` at a propriate time. Note that different combination of modules may have different exploit paths. As long as the above attack vectors remain, the setToken is vulnerable.

Assume a setToken with `CompoundLeverageModule`, `NotionalTradeModule` and `BasicIssuanceModule` with the following positions: 1. CDAI: 100  2. wfCash-DAI 100  and totalSupply = 100. The community decides to remove the `compoundLeverageModule` from the set token. Since `notionalTradeModule` can handle cDAI, the community vote to just call `removeModule` to remove `compoundLeverageModule`. The exploiter has the time to build an exploit and wait the right timing to come.

0. The exploiter listen the manager multisig wallet.
1. Exploiter issue 10 setToken.
2. During the `_callTokensToSend` of `wfcash`, the totalSupply = 100, CDAI = 110, wfCash-DAI = 110.
3. Call `sync` of `CompoundLeverageModule`. `_getCollateralPosition` get  `_cToken.balanceOf(address(_setToken)) = 110` and `totalSupply = 100` and update the `DefaultUnit` of `CETH` 1,1X.
4. Replay multisig wallet's payload and remove `compoundLeverageModule`.
5. The `setToken` can no longer issue / redeem as it would raise `undercollateralized` error. Further, `setValuer` would give a pumped valuation that may cause harm to other protocols.


## Proof of Concept

[POC](https://gist.github.com/Jonah246/13e58b59765c0334189c99a9f29c6dab)
The exploit is quite lengthy. Please check the `Attack.sol` for the main exploit logic.
```soliditiy
    function register() public {
        _ERC1820_REGISTRY.setInterfaceImplementer(address(this), _TOKENS_SENDER_INTERFACE_HASH, address(this));
        _ERC1820_REGISTRY.setInterfaceImplementer(address(this), _TOKENS_SENDER_INTERFACE_HASH, address(this));
    }

    function attack(uint256 _amount) external {
        cToken.approve(address(issueModule), uint256(-1));
        wfCash.approve(address(issueModule), uint256(-1));
        issueModule.issue(setToken, _amount, address(this));
    }

    function tokensToSend(
        address operator,
        address from,
        address to,
        uint256 amount,
        bytes calldata userData,
        bytes calldata operatorData
    ) external {
        compoundModule.sync(setToken, false);
        manager.removeModule(address(setToken));
    }
```

## Tools Used
Manual inspection.

## Recommended Mitigation Steps

The design choice of wfcash being an `ERC777` seems unnecessary to me. Over the past two years, ERC777 leads to so many exploits. [IMBTC-UNISWAP](https://defirate.com/imbtc-uniswap-hack/) [CREAM-AMP](https://twitter.com/CreamdotFinance/status/1432249771750686721?s=20) I recommend the team using ERC20 instead.

If the SetToken team considers supporting ERC777 necessary, I recommend implementing protocol-wide cross-contract reentrancy prevention. Please refer to Rari-Capital. [Comptroller.sol#L1978-L2002](https://github.com/Rari-Capital/fuse-v1/blob/development/src/core/Comptroller.sol#L1978-L2002)

Note that, `Rari` was [exploited](https://www.coindesk.com/business/2022/04/30/defi-lender-rari-capitalfei-loses-80m-in-hack/) given this reentrancy prevention. Simply making `nonReentrant` cross-contact prevention may not be enough. I recommend to setToken protocol going through every module and re-consider whether it's re-entrancy safe.

