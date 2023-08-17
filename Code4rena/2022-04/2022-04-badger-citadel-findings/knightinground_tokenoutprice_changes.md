## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [KnightingRound tokenOutPrice changes](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/73) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/KnightingRound.sol#L162-L204


# Vulnerability details

## Impact
`Function.buy` buys the tokens for whatever price is set as `tokenOutPrice`. This might lead to accidental collisions or front-running attacks when user is trying to buy the tokens and his transaction is being included after the transaction of changing the price of the token via `setTokenOutPrice`.

Scenario:
1. User wants to `buy` tokens and can see price `tokenOutPrice`
2. User likes the price and issues a transaction to `buy` tokens
3. At the same time `CONTRACT_GOVERNANCE_ROLE` account is increasing `tokenOutPrice` through `setTokenOutPrice`
4. `setTokenOutPrice` transaction is included before user's `buy` transaction
5. User buys tokens with the price he was not aware of

Another variation of this attack can be performed using front-running.

## Proof of Concept
* https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/KnightingRound.sol#L162-L204

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add additional parameter `uint256 believedPrice` to `KnightingRound.buy` function and check if `believedPrice` is equal to `tokenOutPrice`.

