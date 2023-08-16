## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Require statement missing in fallback and burnEtherForMember() functions](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/137) 

# Handle

Reigada


# Vulnerability details

## Impact
The contract BasicSale contains a fallback function and a burnEtherForMember() function with exactly the same implementation. These 2 functions do the following call:
_recordBurn(msg.sender, msg.sender, currentEra, currentDay, msg.value);

The _recordBurn function contains the following if block:
if (mapEraDay_MemberUnits[_era][_day][_member] == 0) {                              // If hasn't contributed to this Day yet
    mapMemberEra_Days[_member][_era].push(_day);                                    // Add it
    mapEraDay_MemberCount[_era][_day] += 1;                                         // Count member
    mapEraDay_Members[_era][_day].push(_member);                                    // Add member
}

What does this mean? If a user performs multiple calls to the contract sending 0 ether as msg.value, the if block will be entered and a new key will be pushed to the mapping. Luckily the cost of an addition to or a read from a mapping does not change with the number of keys mapped. But this would totally mess the function getDaysContributedForEra output. Currently this function is only used as a view function, and not used by the smart contract itself. But it's a risk for future implementations that may make use of it.

## Proof of Concept
>>> user2.transfer(to=publicsale.address, amount=0)
Transaction sent: 0xd65ffecd5052314bc09f616461ff6aa9efeb857151c0339dc15653fb90ebb91f
  Gas price: 0.0 gwei   Gas limit: 6721975   Nonce: 0
  Transaction confirmed   Block: 13577879   Gas used: 117368 (1.75%)

<Transaction '0xd65ffecd5052314bc09f616461ff6aa9efeb857151c0339dc15653fb90ebb91f'>

>>> publicsale.getDaysContributedForEra(user2.address, 1)
1

>>> user2.transfer(to=publicsale.address, amount=0)
Transaction sent: 0x0a48f89c1266af2c3eea5cdcec93dae76e4ed2a0936e53e8713d669989b88b19
  Gas price: 0.0 gwei   Gas limit: 6721975   Nonce: 1
  Transaction confirmed   Block: 13577880   Gas used: 91568 (1.36%)

<Transaction '0x0a48f89c1266af2c3eea5cdcec93dae76e4ed2a0936e53e8713d669989b88b19'>

>>> publicsale.getDaysContributedForEra(user2.address, 1)
2

>>> user2.transfer(to=publicsale.address, amount=0)
Transaction sent: 0xeb85e31664eb1578f54daba1be112f9533948d0e2414510874ed55fbb3a9a9e0
  Gas price: 0.0 gwei   Gas limit: 6721975   Nonce: 2
  Transaction confirmed   Block: 13577881   Gas used: 91568 (1.36%)

<Transaction '0xeb85e31664eb1578f54daba1be112f9533948d0e2414510874ed55fbb3a9a9e0'>

>>> publicsale.getDaysContributedForEra(user2.address, 1)
3

## Tools Used
Manual testing

## Recommended Mitigation Steps
Add the following require statement to the fallback and the burnEtherForMember() functions:
require(msg.value > 0, "Some ether should be sent")

