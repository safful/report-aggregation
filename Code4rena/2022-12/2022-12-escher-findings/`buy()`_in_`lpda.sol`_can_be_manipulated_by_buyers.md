## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-06

# [`buy()` in `LPDA.sol` Can be Manipulated by Buyers](https://github.com/code-423n4/2022-12-escher-findings/issues/280) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L68


# Vulnerability details

## Impact
A buyer could plan on buying early at higher prices to make sure he would secure a portion (say 50%) of NFTs he desired. When the number of NFTs still available got smaller and that `sale.endTime` were yet to hit, he would then watch the mempool and repeatedly attempt to thwart the final group of buyers from successfully completing their respective transactions amidst the efforts to prolong the Dutch auction till `sale.endTime` was reached. 

## Proof of Concept
Assuming this particular edition pertained to a 100 NFT collection that would at most last for 60 minutes, and Bob planned on minting 10 of them. At the beginning of the dutch auction, he would first mint 5 NFTs at higher prices no doubt. At 50th minute, `sale.currentId == 95`. Alice, upon seeing this, made up her mind and proceeded to buying the remaining NFTs. Bob, seeing this transaction queuing in the mempool, invoked `buy()` to mint 1 NFT by sending in higher amount of gas to front run Alice. Needless to say, Alice's transaction was going to revert on line 68 because `newId == 101`:

[File: LPDA.sol#L68](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L68)

```
68:        require(newId <= temp.finalId, "TOO MANY");
```
Noticing the number of NFTs still available had become 4, Alice attempted to mint the remaining 4 NFTs this time. Bob, upon seeing the similar queue in the mempool again, front ran Alice with another mint of 1 NFT. 

These steps were repeatedly carried out until Bob managed to get all NFTs he wanted where the last one was minted right after `sale.endTime` hit. At this point, every successful buyers was happy to get the biggest refund possible ensuring that each NFT was only paid for the lowest price. This intended goal, on the contrary, was achieved at the expense of the seller getting the lowest amount of revenue and that the front run buyers minting nothing. 

## Tools Used
Manual inspection

## Recommended Mitigation Steps
Considering refactoring the affected code line as follows:

```
- require(newId <= temp.finalId, "TOO MANY");
+ if(newId > temp.finalId) {
+      uint256 diff = newId - temp.finalId; 
+      newId = temp.finalId;
+      amountSold -= diff;
+      amount -= diff;
+ }
```