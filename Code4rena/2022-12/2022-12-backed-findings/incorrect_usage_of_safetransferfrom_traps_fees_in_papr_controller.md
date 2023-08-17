## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [Incorrect usage of safeTransferFrom traps fees in Papr Controller](https://github.com/code-423n4/2022-12-backed-findings/issues/110) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382-L384


# Vulnerability details

## Impact
Because the Papr Controller never gives approval for ERC20 transfers, calls to `safeTransferFrom` on the Papr token will revert with insufficient approval. This will trap proceeds from auctions in the contract and prevent the owner/ DAO from collecting fees, motivating the rating of high severity. The root cause of this issue is misusing `safeTransferFrom` to transfer tokens directly out of the contract instead of using `transfer` directly. The contract will hold the token balance and thus does not need approval to transfer tokens, nor can it approve token transfers in the current implementation.

## Proof of Concept
Comment out [this token approval](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/test/paprController/OwnerFunctions.ft.sol#L67) as the controller contract does not implement functionality to call approve. It doesn't make sense to "prank" a contract account in this context because it deviates from the runtime behavior of the deployed contract. That is, it's impossible for the Papr Controller to approve token transfers. Run `forge test -m testSendPaprFromAuctionFeesWorksIfOwner` and observe that it fails because of insufficient approvals. Replace [the call to `safeTransferFrom`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L383) with a call to `transfer(to, amount)` and rerun the test. It will now pass and correctly achieve the intended behavior. 

## Tools Used

Foundry

## Recommended Mitigation Steps
Call `transfer(to, amount)` instead of `safeTrasferFrom` [here](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L383). Note, it's unnecessary to use `safeTransfer` as the Papr token doesn't behave irregularly.