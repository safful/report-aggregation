## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [L1Migrator.migrateLPT` can be used to take away protocol's access to LPT tokens in BridgeMinter](https://github.com/code-423n4/2022-01-livepeer-findings/issues/97) 

# Handle

Ruhum


# Vulnerability details

# Vulnerability details

## Impact
Same thing as the ETH issue I reported earlier. I wasn't sure if those are supposed to be a single issue or not. The concept is the same. But, now you lose LPT tokens.

The `L1Migrator.migrateLPT()` function can be called by **anyone**. It pulls all the LPT from the `BridgeMinter` contract and starts the process of moving the funds to L2. First of all, this function is only executable once. The RetryableTicket created with the first call is the only chance of moving the funds to L2.

The attacker can call the function with [parameters](https://developer.offchainlabs.com/docs/l1_l2_messages#parameters) that make the creation of the RetryableTicket on L2 fail. Thus, the LPT sits in the L1Migrator contract with no way of moving it to L2 or anywhere else. Effectively, the funds are lost.

## Proof of Concept
The function is only executable once because it uses the `amount` returned by `IBridgeMinter(bridgeMinterAddr).withdrawLPTToL1Migrator()` to specify the amount of LPT to be sent to L2: https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L342

After the first call to `migrateLPT()` that function will always return 0 since the `BridgeMinter` won't have any more LPT: https://github.com/livepeer/protocol/blob/streamflow/contracts/token/BridgeMinter.sol#L107

So after the attacker called `migrateLPT()` with insufficient funds to create a RetryableTicket on L2 we have the following state:
- BridgeMinter has 0 LPT
- L1Migrator has X amount of LPT that is not accessible. There are no functions to get the LPT out of there.
- 1 failed RetryTicket

The same thing can also be triggered by a non-malicious caller by simply providing insufficient funds. The whole design of only being able to try once is the issue here.

## Tools Used
none

## Recommended Mitigation Steps
Instead of using the `amount` returned by `IBridgeMinter(bridgeMinterAddr).withdrawLPTToL1Migrator()` you should use the balance of the `L1Migrator` contract.

It might also make sense to **not** allow anybody to call the function. I don't see the benefit of that.

`EDIT` Actually, the funds aren't lost. The funds are sent to the Escrow contract which can be used to transfer the funds back to the BridgeMinter contract. Thus, you could reset the whole thing to its initial state and call `L1Migrator.migrateLPT()` again. But, a really persistent attacker has the ability to DoS the function by frontrunning any call to it which results in the RetryableTicket failing again. Thus, you'd have to transfer the funds from the Escrow contract to the BrigeMinter again and so on.

So the same scenario I've outlined earlier is still viable. It's just a bit more difficult now since it has a higher cost for the attacker now. Because of that I think it's an medium issue instead of high.

Also, the mitigation steps I've given aren't valid. You can't use the `L1Migrator` contract's balance since it will always be 0 (the funds are sent to the Escrow contract). Thus the best solution would be to just limit the access to the function.