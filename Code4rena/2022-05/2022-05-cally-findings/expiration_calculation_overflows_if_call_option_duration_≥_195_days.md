## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Expiration calculation overflows if call option duration ≥ 195 days](https://github.com/code-423n4/2022-05-cally-findings/issues/16) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L238


# Vulnerability details

## Details & Impact

`vault.durationDays` is of type `uint8`, thus allowing a maximum value of 255. `1 days = 86400`, thus fitting into a `uint24`. Solc creates a temporary variable to hold the result of the intermittent multiplication `vault.durationDays * 1 days` using the data type of the larger operand.

In this case, the intermittent data type used would be `uint24`, which has a maximum value of `2**24 - 1 = 16777215`. The maximum number allowable before overflow achieved is therefore `(2**24 - 1) / 86400 = 194`.

## Proof of Concept

Insert this test case into [BuyOption.t.sol](https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/test/units/BuyOption.t.sol)

```solidity

function testCannotBuyDueToOverflow() public {
  vm.startPrank(babe);
  bayc.mint(babe, 2);
  // duration of 195 days
  vaultId = c.createVault(2, address(bayc), premiumIndex, 195, strikeIndex, 0, Cally.TokenType.ERC721);
  vm.stopPrank();

  vm.expectRevert(stdError.arithmeticError);
  c.buyOption{value: premium}(vaultId);
}
```

Then run

```
forge test --match-contract TestBuyOption --match-test testCannotBuyDueToOverflow
```

## Tidbit

This was the 1 high-severity bug that I wanted to mention at the end of the [C4 TrustX showcase](https://youtu.be/up9eqFRLgMQ?t=5722) but unfortunately could not due to a lack of time :( It can be found in the [vulnerable lottery contract](https://gist.github.com/HickupHH3/d214cfe6e4d003f428a63ae7d127af2d) on L39. Credits to Pauliax / Thunder for the recommendation and raising awareness of this bug =p

## Reference

[Article](https://muellerberndt.medium.com/building-a-secure-nft-gaming-experience-a-herdsmans-diary-1-91aab11139dc)

## Recommended Mitigation Steps

Cast the multiplication into `uint32`.

```solidity
vault.currentExpiration = uint32(block.timestamp) + uint32(vault.durationDays) * 1 days;
```

