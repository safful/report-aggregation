## Tags

- bug
- 2 (Med Risk)
- in discussion
- primary issue
- sponsor confirmed
- depositEther OOG

# [frxETHMinter.depositEther may run out of gas, leading to lost ETH](https://github.com/code-423n4/2022-09-frax-findings/issues/17) 

# Lines of code

https://github.com/code-423n4/2022-09-frax/blob/dc6684f77b4e9bd965e8862be7f5fb71473a4c4c/src/frxETHMinter.sol#L129


# Vulnerability details

## Impact
`frxETHMinter.depositEther` always iterates over all deposits that are possible with the current balance (`(address(this).balance - currentWithheldETH) / DEPOSIT_SIZE`). However, when a lot of ETH was deposited into the contract / it was not called in a long time, this loop can reach the gas limit. When this happens, no more calls to `depositEther` are possible, as it will always run out of gas.

Of course, the probability that such a situation arises depends on the price of ETH. For >1,000 USD it would need require someone to deposit a large amount of money (which can also happen, there are whales with thousands of ETH, so if one of them would decide to use frxETH, the problem can arise). For lower prices, it can happen even for small (in dollar terms) deposits. And in general, the correct functionality of a protocol should not depend on the price of ETH.

## Proof Of Concept
Jerome Powell continues to rise interest rates, he just announced the next rate hike to 450%. The crypto market crashes, ETH is at 1 USD. Bob buys 100,000 ETH for 100,000 USD and deposits them into `frxETHMinter`. Because of this deposit, `numDeposit` within `depositEther` is equal to 3125. Therefore, every call to the function runs out of gas and it is not possible to deposit this ETH into the deposit contract.

## Recommended Mitigation Steps
It should be possible to specify an upper limit for the number of deposits such that progress is possible, even when a lot of ETH was deposited into the contract.