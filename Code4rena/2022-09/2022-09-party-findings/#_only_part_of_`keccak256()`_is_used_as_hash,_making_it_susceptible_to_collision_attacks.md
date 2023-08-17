## Tags

- bug
- 2 (Med Risk)
- high quality report
- resolved
- sponsor confirmed

# [# Only part of `keccak256()` is used as hash, making it susceptible to collision attacks](https://github.com/code-423n4/2022-09-party-findings/issues/231) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L275
https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/Crowdfund.sol#L325
https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/distribution/TokenDistributor.sol#L26


# Vulnerability details



At 2 places in the code only part of the output of `keccak256()` is used as the hash:
* At `TokenDistributor` - `DistributionState.distributionHash15` - uses only a 15 bytes as a hash
    * This one is intended to save storage
* At `Crowdfund.governanceOptsHash` a 16 bytes is used as hash
    * This one has no benefit at all as it doesn't save on storage


15/16 bytes hash is already not very high to begin with (as explained below). On top of that, using a non standard hash can be unsafe. Since diverging from the standard can break things.



## Impact


For the `FixedGovernanceOpts` an attacker can create a legitimate party, and then when running `buy()` use the malicious hash to:
* include himself in the hosts (DoS-ing the party by vetoing every vote)
* reduce the `passThresholdBps` (allowing him to pass any vote, including sending funds from the Party)
* Setting himself as `feeRecipient` and increasing the fee


For the `DistributionInfo` struct - an attacker can easily drain all funds from the token distribution contract, by using the legitimate hash to create a distribution with a malicious ERC20 token (and a malicious party contract), and then using the malicious hash to claim assets of a legitimate token.

## Proof of Concept

### The attack


Using the birthday attack, for a 50% chance with a 15 bytes hash, the number of hashes needed to generate is 1.4e18 (`(ln(1/0.5) *2) ** 0.5 * (2 ** 60)`).
* For 16 bytes that would be 2.2e19

An attacker can create 2 different structs, a legitimate and a malicious one, while modifying at each iteration only the last bits
* For the `FixedGovernanceOpts` the last bits would be the `feeRecipient` field
* For the `DistributionInfo` struct that would be the `fee` field (and then exploit it via the `claim()` function which doesn't validate the `fee` field)

The attacker will than generate half of the hashes from the malicious one, and half from the legitimate ones, so in case of a collision there's a 50% chance it'd be between the legitimate and malicious struct.



### CPU

* In the `DistributionInfo` we have 224 bytes (and for `FixedGovernanceOpts` 192 bytes if we precalculate the hosts hash)
* A computer needs about 11 cycles per byte
* An avg home PC can do ~3e9 cycles per seconds
* There are ~8.6e4 seconds a day
* Putting it all together `1.4e18 * 11 * 224 / (3e9*8.6e4)` = ~1.3e8

* Note that we can further optimize it (by 10 times at least), since we're using the same input and only modifying the last bits every time (the `fee` field)

### Storage
32 * 1.4e18 = ~4.5e19 bytes is needed, while an affordable drive can be 8TB=~8e12 bytes.
That puts it about 5e6 times away from and affordable attack.


### Overall Risk
The calculations above are for basic equipment, an attacker can be spending more on equipment to get closer (I'd say you can easily multiply that by 100 for a medium size attacker + running the computations for more than one day)
Combining that with the fact that a non-standard hash is used, and that in general hashes can have small vulnerabilities that lower a bit their strength - I'd argue it's not very safe to be ~1e4 (for a medium size attacker; ~1.5e5 for 16 bytes) away from a practical attack.



## Recommended Mitigation Steps
Use the standard, 32-bytes, output of `keccak256()`