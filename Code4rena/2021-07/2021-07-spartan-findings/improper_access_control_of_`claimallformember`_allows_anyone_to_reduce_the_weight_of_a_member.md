## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Improper access control of `claimAllForMember` allows anyone to reduce the weight of a member](https://github.com/code-423n4/2021-07-spartan-findings/issues/235) 

# Handle

shw


# Vulnerability details

## Impact

The `claimAllForMember` function of `Dao` is permissionless, allowing anyone to claim the unlocked bonded LP tokens for any member. However, claiming a member's LP tokens could decrease the member's weight in the `BondVault`, thus affecting the member's votes and rewards in the `Dao` contract. 

## Proof of Concept

For example, an attacker can intentionally front-run a victim's `voteProposal` call to decrease the victim's vote weight to prevent the proposal from being finalized:

1. Supposing the victim's member weight in the `BondVault` is 201, the total weight is 300. The victim has some LP tokens claimable from the vault, and if claimed, the victim's weight will be decreased to 101. To simplify the situation, assuming that the victim's weight in the `DaoVault` and the total weight of the `DaoVault` are both 0.
2. The victim wants to vote on the current proposal, which requires the majority consensus. If the victim calls `voteProposal`, the proposal will be finalized since the victim has the majority weight (201/300 > 66.6%).
3. An attacker does not want the proposal to be finalized, so he calls `claimAllForMember` with the victim as the parameter to intentionally decrease the victim's weight.
4. As a result, the victim's weight is decreased to 101, and the total weight is decreased to 200. The victim cannot finalize the proposal since he has no majority anymore (101/200 < 66.6%).

Similarly, an attacker can front-run a victim's `harvest` call to intentionally decrease the victim's reward since the amount of reward is calculated based on the victim's current weight.

Referenced code:
[Dao.sol#L179-L206](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L179-L206)
[Dao.sol#L276-L285](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L276-L285)
[Dao.sol#L369-L383](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L369-L383)
[Dao.sol#L568-L574](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L568-L574)
[Dao.sol#L577-L586](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L577-L586)
[BondVault.sol#L104-L117](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/BondVault.sol#L104-L117)
[BondVault.sol#L120-L129](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/BondVault.sol#L120-L129)
[BondVault.sol#L155-L162](https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/BondVault.sol#L155-L162)

## Recommended Mitigation Steps

Consider removing the `member` parameter in the `claimAllForMember` function and replace all `member` to `msg.sender` to allow only the user himself to claim unlocked bonded LP tokens.

