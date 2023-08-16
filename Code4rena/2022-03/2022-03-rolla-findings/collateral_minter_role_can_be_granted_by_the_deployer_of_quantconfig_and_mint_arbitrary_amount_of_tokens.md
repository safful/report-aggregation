## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [COLLATERAL_MINTER_ROLE can be granted by the deployer of QuantConfig and mint arbitrary amount of tokens](https://github.com/code-423n4/2022-03-rolla-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/options/CollateralToken.sol#L101-L117


# Vulnerability details

## Impact
```
    function mintCollateralToken(
        address recipient,
        uint256 collateralTokenId,
        uint256 amount
    ) external override {
        require(
            quantConfig.hasRole(
                quantConfig.quantRoles("COLLATERAL_MINTER_ROLE"),
                msg.sender
            ),
            "CollateralToken: Only a collateral minter can mint CollateralTokens"
        );

        emit CollateralTokenMinted(recipient, collateralTokenId, amount);

        _mint(recipient, collateralTokenId, amount, "");
    }
```

Using the mintCollateralToken() function of CollateralToken, an address with COLLATERAL_MINTER_ROLE can mint an arbitrary amount of tokens.

If the private key of the deployer or an address with the COLLATERAL_MINTER_ROLE is compromised, the attacker will be able to mint an unlimited amount of collateral tokens.

We believe this is unnecessary and poses a serious centralization risk.

## Proof of Concept
https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/options/CollateralToken.sol#L101-L117
https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/options/CollateralToken.sol#L138-L160
## Tools Used
None

## Recommended Mitigation Steps
Consider removing the COLLATERAL_MINTER_ROLE, make the CollateralToken only mintable by the owner, and make the Controller contract to be the owner and therefore the only minter.

