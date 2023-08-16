## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`MochiTreasuryV0.sol` Is Unusable In Its Current State](https://github.com/code-423n4/2021-10-mochi-findings/issues/168) 

# Handle

leastwood


# Vulnerability details

## Impact

`MochiTreasuryV0.sol` interacts with Curve's voting escrow contract to lock tokens for 90 days, where it can be later withdrawn by the governance role. However, `VotingEscrow.vy` does not allow contracts to call the following functions; `create_lock()`, `increase_amount()` and `increase_unlock_time()`. For these functions, `msg.sender` must be an EOA account or an approved smart wallet. As a result, any attempt to lock tokens will fail in `MochiTreasuryV0.sol`.

## Proof of Concept

https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/VotingEscrow.vy#L418
https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/VotingEscrow.vy#L438
https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/VotingEscrow.vy#L455

## Tools Used

Manual code review
Discussions with the Mochi team

## Recommended Mitigation Steps

Consider updating this contract to potentially use another escrow service that enables `msg.sender` to be a contract. Alternatively, this escrow functionality can be replaced with an internal contract which holds `usdm` tokens instead, removing the need to convert half of the tokens to Curve tokens. Holding Curve tokens for a minimum of 90 days may overly expose the Mochi treasury to Curve token price fluctuations.

