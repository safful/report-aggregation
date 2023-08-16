## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Gas costs will likely result in any fees sent to the Splitter being economically unviable to recover.](https://github.com/code-423n4/2022-03-joyn-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L161-L163
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L307
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/RoyaltyVault.sol#L43-L50
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L149-L169


# Vulnerability details

## Impact

Collection owners will likely lose money by claiming fees unless the fees from a single NFT sale outweighs the cost of claiming it (not guaranteed).

## Proof of Concept

Consider a new `Collection` with a `RoyaltyVault` and `Splitter` set and a nonzero mint fee.

When calling `mintToken`, the `_handlePayment` function is called
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L161-L163

This will transfer the minting fee to the `RoyaltyVault` contract.

On each transfer of an NFT within the collection (for instance in the `_mint` call which occurs directly after calling `_handlePayment`), the `Collection` contract will call `sendToSplitter` on the `RoyaltyVault`:
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L307

This function will forward the collection owners' portion of the minting on to the `Splitter` contract but another important thing to note is that we call `Splitter.incrementWindow`.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/royalty-vault/contracts/RoyaltyVault.sol#L43-L50

This results in the fees newly deposited into the `Splitter` contract being held in a separate "window" to the fees from previous or later mints and need to be claimed separately. Remember that this process happens on every NFT sale so the only funds which will be held in this window will be the minting fees for this particular mint.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L149-L169

From this we can see that the `claim` function will only claim the fraction of the fees which are owed to the caller from a single NFT mint.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L112-L142

Note that we can attempt to claim from multiple windows in a single transaction using `claimForAllWindow` but as the name suggests it performs an unbounded loop trying to claim all previous windows (even ones which have already been claimed!) and it is likely that with a new window for every NFT sold this function will exceed the gas limit (consider an 10k token collection resulting in trying to do 10k SSTOREs at 20k gas each.), leaving us to claim each window individually with `claim`.

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/splits/contracts/Splitter.sol#L35-L62

We're then forced to claim the royalties from each NFT sold one by one, having to send huge numbers of calls to `claim` incurring the base transaction cost many times over and performing many ERC20 transfers when we could have just performed one.

Compound on this that this needs to be repeated by everyone included in the split, multiplying the costs of claiming.

Medium risk as it's gas inefficiency to the point of significant value leakage where collection owners will lose a large fraction of their royalties.

## Recommended Mitigation Steps

It doesn't seem like the "window" mechanism does anything except raise gas costs to the extent that it will be very difficult to withdraw fees so it should be removed.

