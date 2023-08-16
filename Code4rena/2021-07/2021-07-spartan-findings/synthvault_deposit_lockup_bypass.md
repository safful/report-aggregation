## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [SynthVault deposit lockup bypass](https://github.com/code-423n4/2021-07-spartan-findings/issues/167) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

The `SynthVault.harvestSingle` function can be used to mint & deposit synths without using a lockup.
An attacker sends `BASE` tokens to the pool and then calls `harvestSingle`.
The inner `iPOOL(_poolOUT).mintSynth(synth, address(this));` call will mint synth tokens to the vault based on the total `BASE` balance sent to the pool, including the attacker's previous transfer.
They are then credited the entire amount to their `weight`.

This essentially acts as a (mint +) deposit without a lock-up period.

## Recommended Mitigation Steps
Sync the pool before sending `BASE` to it through `iRESERVE(_DAO().RESERVE()).grantFunds(reward, _poolOUT);` such that any previous `BASE` transfer is wasted.
This way only the actual reward's weight is increased.

