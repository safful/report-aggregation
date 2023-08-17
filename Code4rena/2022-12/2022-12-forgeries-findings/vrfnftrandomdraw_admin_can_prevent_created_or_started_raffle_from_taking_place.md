## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [VRFNFTRandomDraw admin can prevent created or started raffle from taking place](https://github.com/code-423n4/2022-12-forgeries-findings/issues/101) 

# Lines of code

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L162-L168


# Vulnerability details

## Impact
The admin/owner of `VRFNFTRandomDraw` can `startDraw()` a raffle, including emitting the `SetupDraw` event, but in a way that ensures `fulfillRandomWords()` is never called. For example:
- `keyHash` is not validated within `coordinator.requestRandomWords()`. Providing an invalid `keyHash` will allow the raffle to start but prevent the oracle from actually supplying a random value to determine the raffle result.
    - https://github.com/smartcontractkit/chainlink/blob/00f9c6e41f843f96108cdaa118a6ca740b11df35/contracts/src/v0.8/VRFCoordinatorV2.sol#L407-L409
    - https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L163
- The admin/owner could alternatively ensure that the owner-provided chain.link VRF subscription does not have sufficient funds to pay at the time the oracle attempts to supply random values in `fulfillRandomWords()`.
    - https://github.com/smartcontractkit/chainlink/blob/00f9c6e41f843f96108cdaa118a6ca740b11df35/contracts/src/v0.8/VRFCoordinatorV2.sol#L594-L596

In addition, the owner/admin could simply avoid ever calling `startDraw()` in the first place.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used

## Recommended Mitigation Steps
Depending on the desired functionality with respect to the raffle owner, a successful callback to `fulfillRandomWords()` could be a precondition of the admin/owner reclaiming the reward NFT. This would help ensure the owner does not create raffles that they intend will never pay out a reward.