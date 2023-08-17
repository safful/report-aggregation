## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [Non-Deterministic Nonce Leading to Non Cancellable Orders](https://github.com/code-423n4/2022-06-infinity-findings/issues/66) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L375-L402
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L507-L530


# Vulnerability details

## Impact
The current nonce is non-deterministic. This means that at any point in time, it is not possible to say what the current nonce for a given signer is.

**So, there is no way to cancel all pending orders.**

## Proof of Concept
Let's say there was a mass phishing attack on users and some signatures were phished.

Since the user does not know the nonce of the signatures that were signed by him/her, he/she cannot cancel it via `cancelMultipleOrders`.

And there is no way to cancel all pending orders. Since the nonce can be any arbitrary value, if the signature had a UINT256 MAX value, there is no way to cancel it via `cancelAllOrders`.

## Tools Used
VS Code

## Recommended Mitigation Steps
It is recommended to have a deterministic nonce, instead of a non-deterministic nonce.
1. Instead of tracking minNonce, we should track currentNonce.
2. Only allow one increment of the nonce. `cancelAllOrders` should increment the current nonce by 1.
3. Only signatures that match the currentNonce should be considered valid (exactly equal).
4. Instead of canceling orders by nonce, the hash map should be by order hash. So, `isUserOrderNonceExecutedOrCancelled[msg.sender][orderHash]` should be recorded.

If these steps are taken, any such phishing attack can be prevented by simply calling `cancelAllOrders` and incrementing the nonce, because the signature must match the current nonce, and so if the current nonce is incremented by 1, there will be a mismatch and will invalid all pending orders.

Also, specific listings can be canceled by `cancelMultipleOrders` which can check `isUserOrderNonceExecutedOrCancelled[msg.sender][orderHash]`

