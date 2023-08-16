## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Vault assets can be migrated to an arbitrary address at anytime by owner](https://github.com/code-423n4/2021-06-gro-findings/issues/59) 

# Handle

0xRajeev


# Vulnerability details

## Impact

BaseVaultAdaptor contains logic that is “built on top of any vault in order for it to function with Gro protocol.” One of such functions is the migrate() function which is onlyOwner and takes an address parameter which allows owner to migrate vault’s entire balance at any time to that address. This is extremely risky because it gives an opportunity for, at least a perception of, rug-pull by a disgruntled/malicious owner/dev to the protocol users/community. This could also be dangerous if triggered accidentally especially by an EOA owner address or maliciously via compromised keys.

Scenario1: Protocol launches and starts accumulating TVL. A savvy user analyzes source and shares the presence of this migrate() function as potential owner rug-pull vector. Users withdraw funds and protocol reputation takes a hit.

Scenario 2: Protocol launches and hits 100MM TVL. A disgruntled dev/owner migrates vault assets to their address and drains the protocol.

Scenario 3: Protocol launches and hits 100MM TVL. Owner EOA keys get compromised and attacker migrates vault assets to their address and drains the protocol.


## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/vaults/BaseVaultAdaptor.sol#L294-L302

See similar concern on migrate() functionality in ShibaSwap recently:
Yearn dev
https://twitter.com/bantg/status/1412370758987354116
https://twitter.com/bantg/status/1412388385663164425
Others
https://twitter.com/valentinmihov/status/1412352490918625280
https://twitter.com/shegenerates/status/1412642215537545218


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate the need for this function and avoid/mitigate risk appropriately.

