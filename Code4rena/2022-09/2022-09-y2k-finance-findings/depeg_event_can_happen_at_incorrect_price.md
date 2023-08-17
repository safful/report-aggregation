## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- selected for report

# [Depeg event can happen at incorrect price](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/69) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Controller.sol#L96


# Vulnerability details

## Impact
Depeg event can still happen when the price of a pegged asset is equal to the strike price of a Vault which is incorrect. 

This docs clearly mentions:

"When the price of a pegged asset is below the strike price of a Vault, a Keeper(could be anyone) will trigger the depeg event and both Vaults(hedge and risk) will swap their total assets with the other party." - https://code4rena.com/contests/2022-09-y2k-finance-contest

## Proof of Concept

1. Assume strike price of vault is 1 and current price of pegged asset is also 1

2. User calls [triggerDepeg](https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Controller.sol#L148) function which calls isDisaster modifier to check the depeg eligibility

3. Now lets see [isDisaster](https://github.com/code-423n4/2022-09-y2k-finance/blob/main/src/Controller.sol#L83) modifier

```
modifier isDisaster(uint256 marketIndex, uint256 epochEnd) {
        address[] memory vaultsAddress = vaultFactory.getVaults(marketIndex);
        if(
            vaultsAddress.length != VAULTS_LENGTH
            )
            revert MarketDoesNotExist(marketIndex);

        address vaultAddress = vaultsAddress[0];
        Vault vault = Vault(vaultAddress);

        if(vault.idExists(epochEnd) == false)
            revert EpochNotExist();

        if(
            vault.strikePrice() < getLatestPrice(vault.tokenInsured())
            )
            revert PriceNotAtStrikePrice(getLatestPrice(vault.tokenInsured()));

        if(
            vault.idEpochBegin(epochEnd) > block.timestamp)
            revert EpochNotStarted();

        if(
            block.timestamp > epochEnd
            )
            revert EpochExpired();
        _;
    }
```

4. Assume block.timestamp is at correct timestamp (between idEpochBegin and epochEnd), so none of revert execute. Lets look into the interesting one at

```
        if(
            vault.strikePrice() < getLatestPrice(vault.tokenInsured())
            )
            revert PriceNotAtStrikePrice(getLatestPrice(vault.tokenInsured()));
```

5. Since in our case price of vault=price of pegged asset so if condition does not execute and finally isDisaster completes without any revert meaning go ahead of depeg

6. But this is incorrect since price is still not below strike price and is just equal

## Recommended Mitigation Steps
Change the isDisaster modifier to revert when price of a pegged asset is equal to the strike price of a Vault

```
if(
            vault.strikePrice() <= getLatestPrice(vault.tokenInsured())
            )
            revert PriceNotAtStrikePrice(getLatestPrice(vault.tokenInsured()));
```