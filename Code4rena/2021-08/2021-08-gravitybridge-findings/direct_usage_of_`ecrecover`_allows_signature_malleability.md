## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Direct usage of `ecrecover` allows signature malleability](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/61) 

# Handle

shw


# Vulnerability details

## Impact

The `verifySig` function of `Gravity` calls the Solidity `ecrecover` function directly to verify the given signatures. However, the `ecrecover` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible here since the nonce is increased each time, ensuring the signatures are not malleable is considered a best practice (and so is checking `_signer != address(0)`, where `address(0)` means an invalid signature).

## Proof of Concept

Referenced code:
[Gravity.sol#L153](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L153)

[SWC-117: Signature Malleability](https://swcregistry.io/docs/SWC-117)
[SWC-121: Missing Protection against Signature Replay Attacks](https://swcregistry.io/docs/SWC-121)

## Recommended Mitigation Steps

Use the `recover` function from [OpenZeppelin's ECDSA library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) for signature verification.

