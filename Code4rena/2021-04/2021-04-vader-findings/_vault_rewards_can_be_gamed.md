## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed
- filed

# [ Vault rewards can be gamed](https://github.com/code-423n4/2021-04-vader-findings/issues/222) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `_deposit` function increases the member's _weight_ by `_weight = iUTILS(UTILS()).calcValueInBase(iSYNTH(_synth).TOKEN(), _amount);` which is the swap output amount when trading the deposited underlying synth amount.

Notice that anyone can create synths of custom tokens by calling `Pools.deploySynth(customToken)`.

Therefore an attacker can deposit valueless custom tokens and inflate their member weight as follows:

1. Create a custom token and issue lots of tokens to the attacker
2. Create synth of this token
3. Add liquidity for the `TOKEN <> BASE` pair by providing a single wei of `TOKEN` and `10^18` BASE tokens. This makes the `TOKEN` price very expensive.
4. Mint some synths by paying BASE to the pool
5. Deposit the fake synth, `_weight` will be very high because the token pool price is so high.

Call `harvest(realSynth)` with a synth with actual value. This will increase the synth balance and it can be withdrawn later.

## Impact

Anyone can inflate their member weight through depositing a custom synth and earn almost all vault rewards by calling `harvest(realSynth)` with a valuable "real" synth.
The rewards are distributed pro rata to the member weight which is independent of the actual synth deposited.

## Recommended Mitigation Steps

The `calcReward` function completely disregards the `synth` parameter which seems odd.
Think about making the rewards based on the actual synths deposited instead of a "global" weight tracker.
Alternatively, whitelist certain synths that count toward the weight, or don't let anyone create synths.


