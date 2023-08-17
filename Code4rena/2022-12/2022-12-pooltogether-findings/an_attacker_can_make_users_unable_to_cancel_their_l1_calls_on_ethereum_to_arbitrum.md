## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- M-01

# [An attacker can make users unable to cancel their L1 calls on Ethereum To Arbitrum](https://github.com/code-423n4/2022-12-pooltogether-findings/issues/60) 

# Lines of code

https://github.com/pooltogether/ERC5164/blob/5647bd84f2a6d1a37f41394874d567e45a97bf48/src/ethereum-arbitrum/EthereumToArbitrumRelayer.sol#L118-#L127


# Vulnerability details

## Impact
An attacker can make users unable to cancel their L1 calls on Ethereum To Arbitrum.

## Proof of Concept
When someone want to make  calls to Arbitrum from Ethereum, first they call `relayCalls` to fingerprint their data and then anyone else can call `processCalls` to process the calls. According to the doc in Inbox source code https://github.com/OffchainLabs/nitro/blob/1f32bec6b9b228bb2fab4bfa02867716f65d0c5c/contracts/src/bridge/Inbox.sol#L427, function `createRetryableTicket` has one parameter called `callValueRefundAddress` and this is the address that is granted the option to `cancel` a `Retryable`. In `EthereumToArbitrumRelayer.sol` it's currently set as `msg.sender` (5th parameter) which is whoever make the call to  function `processCall`:

```
uint256 _ticketID = inbox.createRetryableTicket{ value: msg.value }(
      address(executor),
      0,
      _maxSubmissionCost,
      msg.sender,
      msg.sender,
      _gasLimit,
      _gasPriceBid,
      _data
    );
```
This implementation allows an attacker to remove the possibility of a user to cancel their calls, which is an important mechanism to be properly implemented. This scenario demonstrates how this could happen:
- User A call `relayCalls` to fingerprint their calls
- User B call `processCalls` to process user A's calls.
- User A now changes his mind and wants to cancel his calls but he's unable to since  `callValueRefundAddress` is set to user B's address, now user B is the one who decides whether to cancel user A's calls or not, which should be user A's option.
- Another common case is when users's calls failed, anyone can try to `redeem` it, according to the doc https://developer.arbitrum.io/arbos/l1-to-l2-messaging. So if a someone calls `processCalls` to process others's calls and it fails, the owner of the calls now cannot cancel their calls and anyone else can redeem (reexecute) them.

It should be noted here that `EthereumToArbitrumRelayer.sol` provides no other functionality to cancel users's calls, but it seems to rely only on Arbitrum's Retryable cancel mechanism to do so. 

## Tools Used
Manual review.

## Recommended Mitigation Steps
Currently, anyone can process others's calls by calling `processCalls` functions and I think this does not pose any security risk as long as the user who actually fingerprinted these calls can reserve their rights to cancel it if they want to. Therefore, I recommend changing `callValueRefundAddress` in `createRetryableTicket` to `_sender`, this combines with event `ProcessedCalls(_nonce, msg.sender, _ticketID)` emitted at the end of `processCalls` function will allow a user to be notified if their calls has been processed by anyone else and they can cancel it in L2 using `_ticketID`.
