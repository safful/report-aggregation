## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-09

# [GroupBuy can be drained of all ETH.](https://github.com/code-423n4/2022-12-tessera-findings/issues/52) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L204-L219


# Vulnerability details

## Description


purchase() in GroupBuy faciilitates the purchasing of an NFT after enough contributions were gathered. Another report titled *"Attacker can steal the amount collected so far in the GroupBuy for NFT purchase*" describes a high impact bug in purchase. It is advised to read that first for context.

Additionally, purchase() is vulnerable to a re-entrancy exploit which can be *chained* or *not chained* to the \_market issue to steal *the entire* ETH stored in GroupBuy, rather than being capped to `minReservePrices[_poolId] * filledQuantities[_poolId]`. 

Attacker may take control of execution using this call:
```
// Executes purchase order transaction through market buyer contract and deploys new vault
address vault = IMarketBuyer(_market).execute{value: _price}(_purchaseOrder);
```
It could occur either by exploiting the unvalidated \_market vulnerability , or by abusing an existing market that uses a user address in \_purchaseOrder. 

There is no re-entrancy protection in purchase() call:
```
function purchase(
    uint256 _poolId,
    address _market,
    address _nftContract,
    uint256 _tokenId,
    uint256 _price,
    bytes memory _purchaseOrder,
    bytes32[] memory _purchaseProof
) external {
```

\_verifyUnsuccessfulState() needs to not revert for purchase call. It checks the pool.success flag:
`if (pool.success || block.timestamp > pool.terminationPeriod) revert InvalidState();`

However, success is only set as the last thing in purchase():
```
    // Stores mapping value of poolId to newly deployed vault
    poolToVault[_poolId] = vault;
    // Sets pool state to successful
    poolInfo[_poolId].success = true;
    // Emits event for purchasing NFT at given price
    emit Purchase(_poolId, vault, _nftContract, _tokenId, _price);
}
```

Therefore, attacker can re-enter purchase() function multiple times, each time extracting the maximum allowed price. If attacker uses the controlled \_market exploit, the function will return the current NFT owner, so when all the functions unwind they will keep setting success to true and exit nicely.

## Impact

GroupBuy can be drained of all ETH.

## Proof of Concept

1. GroupBuy holds 1500 ETH, from various bids
2. maximum allowed price (`minReservePrices[_poolId] * filledQuantities[_poolId]`) is 50 * 20 = 1000 ETH
3. purchase(1000 ETH) is called
	1. GroupBuy sends attacker 1000 ETH and calls execute()
		1. execute() calls purchase(500ETH)
			1. GroupBuy sends attacker 500 ETH and calls execute()
				1. execute returns NFT owner address
			2. GroupBuy sees returned address is NFT owner. Marks success and returns
		2. execute returns NFT owner address
	2. GroupBuy sees returned address is NFT owner. Marks success and returns
4. Attacker is left with 1500 ETH. Previous exploit alone can only net 1000ETH. Additionally, this exploit can be chained to any trusted MarketBuyer which passes control to user for purchasing and storing in vault, and then returns a valid vault.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Add a re-entrancy guard to purchase() function. Also, change success variable before performing external contract calls.