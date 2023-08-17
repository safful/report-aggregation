## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- upgraded by judge
- H-02

# [Draw organizer can rig the draw to favor certain participants such as their own account.](https://github.com/code-423n4/2022-12-forgeries-findings/issues/272) 

# Lines of code

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L83


# Vulnerability details

## Description

In RandomDraw, the host initiates a draw using startDraw() or redraw() if the redraw draw expiry has passed. Actual use of Chainlink oracle is done in \_requestRoll:
```
request.currentChainlinkRequestId = coordinator.requestRandomWords({
    keyHash: settings.keyHash,
    subId: settings.subscriptionId,
    minimumRequestConfirmations: minimumRequestConfirmations,
    callbackGasLimit: callbackGasLimit,
    numWords: wordsRequested
});
```

Use of subscription API is explained well [here](https://docs.chain.link/vrf/v2/subscription). Chainlink VRFCoordinatorV2 is called with requestRandomWords() and emits a random request. After `minimumRequestConfirmations` blocks, an oracle VRF node replies to the coordinator with a provable random, which supplies the random to the requesting contract via `fulfillRandomWords()`  call. It is important to note the role of subscription ID. This ID maps to the subscription charged for the request, in LINK tokens. In our contract, the raffle host supplies their subscription ID as a parameter. Sufficient balance check of the request ID is not [checked](https://github.com/smartcontractkit/chainlink/blob/286a65065fcfa5e1b2362745079cdc218e40e68d/contracts/src/v0.8/VRFCoordinatorV2.sol#L370) at request-time, but rather checked in Chainlink [node](https://github.com/smartcontractkit/chainlink/blob/806ee17236ba70926a1f07d1141808b634db48b6/core/services/vrf/listener_v2.go#L346) code as well as on-chain by [VRFCoordinator](https://github.com/smartcontractkit/chainlink/blob/286a65065fcfa5e1b2362745079cdc218e40e68d/contracts/src/v0.8/VRFCoordinatorV2.sol#L594) when the request is satisfied. In the scenario where the subscriptionID lacks funds, there will be a period of 24 hours when user can top up the account and random response will be [sent](https://docs.chain.link/vrf/v2/subscription):

"Each subscription must maintain a minimum balance to fund requests from consuming contracts. If your balance is below that minimum, your requests remain pending for up to 24 hours before they expire. After you add sufficient LINK to a subscription, pending requests automatically process as long as they have not expired."

The reason this is extremely interesting is because as soon as redraws are possible, the random response can no longer be treated as fair. Indeed, Draw host can wait until redraw cooldown passed (e.g. 1 hour), and only then fund the subscriptionID. At this point, Chainlink node will send a TX with the random response. If host likes the response (i.e. the draw winner), they will not interfere. If they don't like the response, they can simply frontrun the Chainlink TX with a redraw() call. A redraw will create a new random request and discard the old requestId so the previous request will never be accepted.
```
function fulfillRandomWords(
    uint256 _requestId,
    uint256[] memory _randomWords
) internal override {
    // Validate request ID
	  // <---------------- swap currentChainlinkRequestId --->
    if (_requestId != request.currentChainlinkRequestId) {
        revert REQUEST_DOES_NOT_MATCH_CURRENT_ID();
    }
	...
}
```

```
//<------ redraw swaps currentChainlinkRequestId --->
request.currentChainlinkRequestId = coordinator.requestRandomWords({
    keyHash: settings.keyHash,
    subId: settings.subscriptionId,
    minimumRequestConfirmations: minimumRequestConfirmations,
    callbackGasLimit: callbackGasLimit,
    numWords: wordsRequested
});
```

Chainlink docs [warn](https://docs.chain.link/vrf/v2/security) against this usage pattern of the VRF -"Don’t accept bids/bets/inputs after you have made a randomness request". In this instance, a low subscription balance allows the host to invalidate the assumption that 1 hour redraw cooldown is enough to guarantee Chainlink answer has been received.

## Impact

Draw organizer can rig the draw to favor certain participants such as their own account.

## Proof of Concept

Owner offers a BAYC NFT for holders of their NFT collection X. Out of 10,000 tokenIDs, owner has 5,000 Xs. Rest belong to retail users. 
1. owner subscriptionID is left with 0 LINK balance in coordinator
2. redraw time is set to 2 hours
3. owner calls startDraw() which will initiate a Chainlink request
4. owner waits for 2 hours and then tops up their subscriptionID with sufficient LINK
5. owner scans the mempool for fulfillRandomWords()
6. If the raffle winner is tokenID < 5000, it is owner's token
	1. Let fulfill execute and pick up the reward
7. If tokenID >= 5000
	1. Call redraw()
	2. fulfill will revert because of requestId mismatch
8. Owner has 75% of claiming the NFT instead of 50%

Note that Forgeries draws are presumably intended as incentives for speculators to buy NFTs from specific collections. Without having a fair shot at receiving rewards from raffles, these NFTs user buys could be worthless. Another way to look at it is that the impact is theft of yield, as host can freely decrease the probability that a token will be chosen for rewards with this method.

Also, I did not categorize it as centralization risk as the counterparty is not Forgeries but rather some unknown third-party host which offers an NFT incentive program. It is a similar situation to the distinction made between 1st party and 3rd party projects [here](https://github.com/code-423n4/2022-10-juicebox-findings/issues/191)

## Tools Used

Manual audit
[Chainlink docs](https://docs.chain.link/vrf/v2/subscription)
[Chainlink co-ordinator code](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/VRFCoordinatorV2.sol)

## Recommended Mitigation Steps

The root cause is that Chainlink response can arrive up to 24 hours from the most request is dispatched, while redraw cooldown can be 1 hour+. The best fix would be to enforce minimum cooldown of 24 hours.