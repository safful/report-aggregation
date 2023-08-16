## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Win all relayer rewards](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/7) 

# Handle

nascent


# Vulnerability details

"Large Validator Sets/Rapid Validator Set Updates May Freeze the Bridge or Relayer" can affect just the relayers & not affect the oracle in certain circumstances. This could result in valid attestations, but prevent any of the other relayers from being able to participate in the execution. While the other relayers are down from the other attack, the attacker can win all batch, logic, and valset rewards as their node is the only relayer running. This is possible because `find_latest_valset` is run in the main relayer loop and everytime tries for 5000 blocks of logs.

