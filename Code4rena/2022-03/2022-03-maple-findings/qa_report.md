## Tags

- bug
- QA (Quality Assurance)
- disagree with severity
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-03-maple-findings/issues/21) 

# QA Report

## Non-Critical Findings

### Redundant type cast to `address`

#### Description

Variable `asset` is defined as `address public override asset`, the type casting to `address` is redundant.

#### Findings

[RevenueDistributionToken.sol#L162](https://github.com/maple-labs/revenue-distribution-token/blob/v1.0.0-beta.1/contracts/RevenueDistributionToken.sol#L162)  
[RevenueDistributionToken.sol#L181](https://github.com/maple-labs/revenue-distribution-token/blob/v1.0.0-beta.1/contracts/RevenueDistributionToken.sol#L181)

#### Recommended mitigation steps

Remove redundant type cast.

From:

```solidity
require(ERC20Helper.transfer(address(asset), receiver_, assets_), "RDT:B:TRANSFER");
```

To:

```solidity
require(ERC20Helper.transfer(asset, receiver_, assets_), "RDT:B:TRANSFER");
```

---

### Open TODOs in code

#### Description

Open TODOs can hint at programming or architectural errors that still need to be fixed.

#### Findings

[RevenueDistributionToken.sol#L78](https://github.com/maple-labs/revenue-distribution-token/blob/v1.0.0-beta.1/contracts/RevenueDistributionToken.sol#L78)  
[RevenueDistributionToken.sol#L276](https://github.com/maple-labs/revenue-distribution-token/blob/v1.0.0-beta.1/contracts/RevenueDistributionToken.sol#L276)  

#### Recommended mitigation steps

Implement open TODOs and remove comments.

### `xMPL.performMigration()` is safe to be called by everyone

#### Description

`xMPL.performMigration()` is currently only allowed to be called by the contract owner, but as there are no funds at risk and no downsides to having everyone (public) call the function, the modifier `onlyOwner` can be removed.

It even states so in the [README](https://github.com/maple-labs/xMPL/blob/v1.0.0-beta.1/README.md):

```
3. After the time delay, anyone can call `performMigration`, which executes the migration with the parameters set 10 days prior.
```

#### Findings

[xMPL.performMigration()](https://github.com/maple-labs/xMPL/blob/v1.0.0-beta.1/contracts/xMPL.sol#L51)

#### Recommended mitigation steps

Remove modifier `onlyOwner`.

## Low Risk

None found.
