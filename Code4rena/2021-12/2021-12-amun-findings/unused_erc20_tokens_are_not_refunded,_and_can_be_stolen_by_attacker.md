## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unused ERC20 tokens are not refunded, and can be stolen by attacker](https://github.com/code-423n4/2021-12-amun-findings/issues/201) 

# Handle

WatchPug


# Vulnerability details

Under certain circumstances, e.g. `annualizedFee` being minted to `feeBeneficiary` between the time user sent the transaction and the transaction being packed into the block and causing amounts of underlying tokens for each basketToken to decrease. It's possible or even most certainly that there will be some leftover basket underlying tokens, as `BasketFacet.sol#joinPool()` will only transfer required amounts of basket tokens from Join contracts.

However, in the current implementation, only the leftover inputToken is returned.

As a result, the leftover underlying tokens won't be returned to the user, which constitutes users' fund loss.

https://github.com/code-423n4/2021-12-amun/blob/cf890dedf2e43ec787e8e5df65726316fda134a1/contracts/basket/contracts/singleJoinExit/SingleTokenJoinV2.sol#L57-L78

```solidity
function joinTokenSingle(JoinTokenStructV2 calldata _joinTokenStruct)
    external
{
    // ######## INIT TOKEN #########
    IERC20 inputToken = IERC20(_joinTokenStruct.inputToken);

    inputToken.safeTransferFrom(
        msg.sender,
        address(this),
        _joinTokenStruct.inputAmount
    );

    _joinTokenSingle(_joinTokenStruct);

    // ######## SEND TOKEN #########
    uint256 remainingIntermediateBalance = inputToken.balanceOf(
        address(this)
    );
    if (remainingIntermediateBalance > 0) {
        inputToken.safeTransfer(msg.sender, remainingIntermediateBalance);
    }
}
```

https://github.com/code-423n4/2021-12-amun/blob/cf890dedf2e43ec787e8e5df65726316fda134a1/contracts/basket/contracts/facets/Basket/BasketFacet.sol#L143-L168

```solidity
function joinPool(uint256 _amount, uint16 _referral)
    external
    override
    noReentry
{
    require(!this.getLock(), "POOL_LOCKED");
    chargeOutstandingAnnualizedFee();
    LibBasketStorage.BasketStorage storage bs =
        LibBasketStorage.basketStorage();
    uint256 totalSupply = LibERC20Storage.erc20Storage().totalSupply;
    require(
        totalSupply.add(_amount) <= this.getCap(),
        "MAX_POOL_CAP_REACHED"
    );

    uint256 feeAmount = _amount.mul(bs.entryFee).div(10**18);

    for (uint256 i; i < bs.tokens.length; i++) {
        IERC20 token = bs.tokens[i];
        uint256 tokenAmount =
            balance(address(token)).mul(_amount.add(feeAmount)).div(
                totalSupply
            );
        require(tokenAmount != 0, "AMOUNT_TOO_SMALL");
        token.safeTransferFrom(msg.sender, address(this), tokenAmount);
    }
    ...
```

Furthermore, the leftover tokens in the `SingleTokenJoinV2` contract can be stolen by calling `joinTokenSingle()` with fake `outputBasket` contract and `swap.exchange` contract.

### Recomandation

Consider changing to: 

1. Calling `IBasketFacet.calcTokensForAmount()` first and only swap for exactly the desired amounts of tokens (like `SingleTokenJoin.sol`);
2. Or, refund leftover tokens.

