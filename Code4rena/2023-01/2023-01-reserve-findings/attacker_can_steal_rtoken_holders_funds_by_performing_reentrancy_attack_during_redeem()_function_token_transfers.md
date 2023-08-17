## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [attacker can steal RToken holders funds by performing reentrancy attack during redeem() function token transfers](https://github.com/code-423n4/2023-01-reserve-findings/issues/347) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/RToken.sol#L439-L514
https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/BackingManager.sol#L105-L150


# Vulnerability details

## Impact
Function `redeem()` redeems RToken for basket collateral and it updated `basketsNeeded` and transfers users basket ERC20 from BackingManager to user address. it loops through tokens and transfer them to caller and if one of tokens were ERC777 or any other 3rd party protocol token with hook, attacker can perform reentrancy attack during token transfers. Attacker can cause multiple impacts by choosing the reentrancy function:
1. attacker can call `redeem()` again and bypass "bounding each withdrawal by the prorata share when protocol is under-collateralized" because tokens balance of BackingManager is not updated yet.
2. attacker can call `BackingManager.manageTokens()` and because `basketsNeeded` gets decreased and basket tokens balances of BasketManager are not updated, code would detect those tokens as excess funds and would distribute them between RSR stakers and RToken holders and some of RToken deposits would get transferred to RSR holders as rewards.

## Proof of Concept
This is `redeem()` code:
```
    function redeem(uint256 amount) external notFrozen {
...............
...............
        (address[] memory erc20s, uint256[] memory amounts) = basketHandler.quote(baskets, FLOOR);

        uint256 erc20length = erc20s.length;

        uint192 prorate = uint192((FIX_ONE_256 * amount) / supply);

        // Bound each withdrawal by the prorata share, in case we're currently under-collateralized
        for (uint256 i = 0; i < erc20length; ++i) {
            uint256 bal = IERC20Upgradeable(erc20s[i]).balanceOf(address(backingManager));

            uint256 prorata = (prorate > 0)
                ? (prorate * bal) / FIX_ONE // {qTok} = D18{1} * {qTok} / D18
                : mulDiv256(bal, amount, supply); // {qTok} = {qTok} * {qRTok} / {qRTok}

            if (prorata < amounts[i]) amounts[i] = prorata;
        }

        basketsNeeded = basketsNeeded_ - baskets;
        emit BasketsNeededChanged(basketsNeeded_, basketsNeeded);

        // == Interactions ==
        _burn(redeemer, amount);

        bool allZero = true;
        for (uint256 i = 0; i < erc20length; ++i) {
            if (amounts[i] == 0) continue;
            if (allZero) allZero = false;

            IERC20Upgradeable(erc20s[i]).safeTransferFrom(
                address(backingManager),
                redeemer,
                amounts[i]
            );
        }

        if (allZero) revert("Empty redemption");
    }
```
As you can see code calculates withdrawal amount of each basket erc20 tokens by calling `basketHandler.quote()` and then bounds each withdrawal by the prorata share of token balance, in case protocol is under-collateralized. and then code updates `basketsNeeded` and in the end transfers the tokens.  if one of those tokens were ERC777 then that token would call receiver hook function in token transfer. there may be other 3rd party protocol tokens that calls registered hook functions during the token transfer. as reserve protocol is permission less and tries to work with all tokens so the external call in the token transfer can call hook functions. attacker can use this hook and perform reentrancy attack.
This is `fullyCollateralized()` code in BasketHandler:
```
    function fullyCollateralized() external view returns (bool) {
        return basketsHeldBy(address(backingManager)) >= rToken.basketsNeeded();
    }
```
As you can see it calculates baskets that can be held by backingManager tokens balance and needed baskets by RToken contract and by comparing them determines that if RToken is fully collateralized or not. if RToken is fully collateralized then `BackingManager.manageTokens()` would call `handoutExcessAssets()` and would distributes extra funds between RToken holders and RSR stakers.
the root cause of the issue is that during tokens transfers in `redeem()` not all the basket tokens balance of the BackingManager updates once and if one has hook function which calls attacker contract then attacker can use this updated token balance of the contract and perform his reentrancy attack. attacker can call different functions for reentrancy. these are two scenarios:
** scenario #1: attacker call `redeem()` again and bypass prorata share bound check when protocol is under-collaterialized:
1. tokens [`SOME_ERC777`, `USDT`] with quantity [1, 1] are in the basket right now and basket nonce is BasketNonce1.
2. BackingManager has 200K `SOME_ERC777` balance and 100K `USDT` balance. `basketsNeeded` in RToken is 150K and RToken supply is 150K and attacker address Attacker1 has 30k RToken. battery charge allows for attacker to withdraw 30K tokens in one block.
3. attacker would register a hook for his address in `SOME_ERC777` token to get called during transfers.
4. attacker would call `redeem()` to redeem 15K RToken and code would updated `basketsNeeded` to 135K and code would bounds withdrawal by prorata shares of balance of the BackingManager because protocol is under-collateralized and code would calculated withdrawal amouns as 15K `SOME_ERC777` tokens and 10K `USDT` tokens (instead of 15K `USDT` tokens) for withdraws.
5. then contract would transfer 15K `SOME_ERC777` tokens first to attacker address and attacker contract would get called during the hook function and now `basketsNeeded` is 135K and total RTokens is 135K and BackingManager balance is 185K `SOME_ERC777` and 100K `USDT` (`USDT` is not yet transferred). then attacker contract can call `redeem()` again for the remaining 15K RTokens.
6. because protocol is under-collateralized code would calculated withdrawal amouns as 15K `SOME_ERC777` and 11.1K `USDT` (USDT_balance * rtokenAmount / totalSupply = 100K * 15K / 135K) and it would burn 15K RToken form caller and the new value of totalSupply of RTokens would be 120K and `basketsNeeded` would be 120K too. then code would transfers 15K `SOME_ERC777` and 11.1K `USDT` for attacker address. 
7. attacker's hook function would return and `redeem()` would transfer 10K `USDT` to attacker in the rest of the execution. attacker would receive 30K `SOME_ERC777` and 21.1K `USDT` tokens for 15K redeemed RToken but attacker should have get (`100 * 30K / 150K = 20K`) 20K `USDT` tokens because of the bound each withdrawal by the prorata share, in case we're currently under-collateralized.
8. so attacker would be able to bypass the bounding check and withdraw more funds and stole other users funds. the attack is more effective if withdrawal battery charge is higher but in general case attacker can perform two withdraw each with about `charge/2` amount of RToken in each block and stole other users funds when protocol is under collaterlized.

** scenario #2: attacker can call `BackingManager.manageTokens()` for reentrancy call:
1. tokens [`SOME_ERC777`, `USDT`] with quantity [1, 1] are in the basket right now and basket nonce is BasketNonce1.
2. BackingManager has 200K `SOME_ERC777` balance and 150K `USDT` balance. `basketsNeeded` in RToken is 150K and RToken supply is 150K and attacker address Attacker1 has 30k RToken. battery charge allows for attacker to withdraw 30K tokens in one block.
3. attacker would register a hook for his address in `SOME_ERC777` token to get called during transfers.
4. attacker would call `redeem()` to redeem 30K RToken and code would updated `basketsNeeded` to 120K and burn 30K RToken and code would calculated withdrawal amounts as 30K `SOME_ERC777` tokens and 30K `USDT` tokens for withdraws.
5. then contract would transfer 30K `SOME_ERC777` tokens first to attacker address and attacker contract would get called during the hook function and now `basketsNeeded` is 120K and total RTokens is 120K and BackingManager balance is 170K `SOME_ERC777` and 150K `USDT` (`USDT` is not yet transferred). then attacker contract can call `BackingManager.manageTokens()`.
6. function `manageTokens()` would calculated baskets can held by BackingManager and it would be higher than 150K and `basketsNeeded` would be 130K and code would consider 60K `SOME_ERC777` and 30K `USDT` tokens as revenue and try to distribute it between RSR stakers and RToken holders. code would mint 30K RTokens and would distribute it.
7. then attacker hook function would return and `redeem()` would transfer 30K `USDT` to attacker address in rest of the execution.
8. so attacker would able to make code to calculate RToken holders backed tokens as revenue and distribute it between RSR stakers and RSR stakers would receive RTokens backed tokens as rewards. the attack is more effective is battery charge is high but in general case attacker can call redeem for battery charge amount and cause those funds to be counted and get distributed to the RSR stakers (according to the rewards distribution rate)

## Tools Used
VIM

## Recommended Mitigation Steps
prevent reading reentrancy attack by central reentrancy guard or by one main proxy interface contract that has reentrancy guard.
or create contract state (similar to basket nonce) which changes after each interaction and check for contracts states change during the call. (start and end of the call)
