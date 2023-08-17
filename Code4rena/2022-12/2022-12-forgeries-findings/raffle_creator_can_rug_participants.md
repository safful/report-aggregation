## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-01

# [Raffle creator can rug participants](https://github.com/code-423n4/2022-12-forgeries-findings/issues/88) 

# Lines of code

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L304
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L127


# Vulnerability details

## Impact
The raffle creator is not required to actually give the NFT away. The NFT that is used for the raffle is transferred to the contract when `startDraw` is executed. Before that, the NFT is in the hands of the creator. This means that he might create a raffle to make users buy NFTs required to participate and then refuse to draw a winner and keep the NFT to himself. Furthermore, he might not even be the owner of NFT in the first place, which he can achieve by flash loaning the NFT in order to pass the `ownerOf` check in `initialize` function.

## Proof of Concept
### Example 1
1. User U creates an NFT collection C
2. He buys a BAYC NFT
3. He creates a raffle with it, and requires `drawingToken` to be from collection C
4. Users buy tokens from his collection C
5. He then refuses to execute `startDraw` function and rather sells the BAYC NFT

### Example 2
1. User U creates an NFT collection C
2. User U uses an NFT flash loan to borrow a very expensive NFT
3. In the same transaction he creates a raffle with this NFT, and requires `drawingToken` to be from collection C
4. The check that he is the owner will pass, because for the duration of the transaction he in fact is
5. Users see that there is a raffle for a very expensive NFT, so they buy tokens C
6. The winner is never drawn, because the creator does not even own the NFT

### Example 3
1. User U has an NFT X
2. He puts X on a sale on some NFT marketplace (which does not require him to lock it in contract)
3. He forgets about it and creates a raffle with it
4. Users buy the tokens necessary for the raffle
5. User U wants to execute the `startDraw` function, but just before it the NFT X is bought from him through the marketplace
6. The winner cannot be drawn

## Recommended Mitigation Steps
Transfer the NFT to the contract at the time of creation of the raffle.  You can do that by approving the factory contract to transfer the token and do the transfer in [`makeNewDraw`](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L43) function between cloning and `initialization`.
```
address newDrawing = ClonesUpgradeable.clone(implementation);
IERC721(settings.token).transferFrom(msg.sender, newDrawing, settings.tokenId);
        // Setup the new drawing
IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
``` 
Remember to remove token transfer from `startDraw` function.

Notice that the creator can still claim NFT after a week, without drawing, by executing `lastResortTimelockOwnerClaimNFT`. To prevent that, I would recommend adding a check in [`lastResortTimelockOwnerClaimNFT`](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L304), if a winner was drawn.
```
 if (!request.hasChosenRandomNumber) {
            revert NEEDS_TO_HAVE_CHOSEN_A_NUMBER();
}
```
So now a user can trust that the NFT is locked in the contract, and it will be claimable only by a winner (or creator if the winner does not claim it). However, there is still no guarantee that the winner will actually be drawn, because the creator has to manually execute `startDraw` function.
To fix this, I would recommend allowing anyone to execute `startDraw` function, so there is no need to rely on the creator. But we would need to limit the time window of when [`startDraw`](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173) can be executed, so users have the time to get tokens before the drawing. That can be done by introducing a new state variable `firstDrawTime`, that acts as a timestamp after which drawing can happen.
```
if(block.timestamp < firstDrawTime) revert CANNOT_DRAW_YET();
```
Notice that now NFT can only be claimed after winner has been drawn. This means that we are depending on ChainLink VRF to be successful. For that reason I would recommend adding a role that has the power to change the VRF subscription or restore the NFT in case where winner is not picked in reasonable time. This role would be given to protocol owner (owner of the factory) / DAO / someone who would be considered as most reliable.
