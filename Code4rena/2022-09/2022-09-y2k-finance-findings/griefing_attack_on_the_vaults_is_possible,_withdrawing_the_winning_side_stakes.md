## Tags

- bug
- 3 (High Risk)
- high quality report
- sponsor confirmed
- old-submission-method
- selected for report

# [Griefing attack on the Vaults is possible, withdrawing the winning side stakes](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/434) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/SemiFungibleVault.sol#L110-L119
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203-L218


# Vulnerability details

*Anyone* can withdraw to `receiver` once the `receiver` is `isApprovedForAll(owner, receiver)`. The funds will be sent to `receiver`, but it will happen whenever an arbitrary `msg.sender` wants. The only precondition is the presence of any approvals.

This can be easily used to sabotage the system as a whole. Say there are two depositors in the hedge Vault, Bob and David, both trust each other and approved each other. Mike the attacker observing the coming end of epoch where no depeg happened, calls the withdraw() for both Bob and David in the last block of the epoch. Mike gained nothing, while both Bob and David lost the payoff that was guaranteed for them at this point.

Setting the severity to be high as this can be routinely used to sabotage the y2k users, both risk and hedge, depriving them from the payouts whenever they happen to be on the winning side. Usual attackers here can be the users from the another side, risk users attacking hedge vault, and vice versa.

## Proof of Concept

isApprovedForAll() in withdrawal functions checks the `receiver` to be approved, not the caller.

SemiFungibleVault's withdraw:

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/SemiFungibleVault.sol#L110-L119

```solidity
    function withdraw(
        uint256 id,
        uint256 assets,
        address receiver,
        address owner
    ) external virtual returns (uint256 shares) {
        require(
            msg.sender == owner || isApprovedForAll(owner, receiver),
            "Only owner can withdraw, or owner has approved receiver for all"
        );
```

Vault's withdraw:

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203-L218

```solidity
    function withdraw(
        uint256 id,
        uint256 assets,
        address receiver,
        address owner
    )
        external
        override
        epochHasEnded(id)
        marketExists(id)
        returns (uint256 shares)
    {
        if(
            msg.sender != owner &&
            isApprovedForAll(owner, receiver) == false)
            revert OwnerDidNotAuthorize(msg.sender, owner);
```

This way anyone at any time can run withdraw from the Vaults whenever owner has some address approved.

## Recommended Mitigation Steps

Consider changing the approval requirement to be for the caller, not receiver:

SemiFungibleVault's withdraw:

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/SemiFungibleVault.sol#L110-L119

```solidity
    function withdraw(
        uint256 id,
        uint256 assets,
        address receiver,
        address owner
    ) external virtual returns (uint256 shares) {
        require(
-           msg.sender == owner || isApprovedForAll(owner, receiver),
+           msg.sender == owner || isApprovedForAll(owner, msg.sender),
            "Only owner can withdraw, or owner has approved receiver for all"
        );
```

Vault's withdraw:

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203-L218

```solidity
    function withdraw(
        uint256 id,
        uint256 assets,
        address receiver,
        address owner
    )
        external
        override
        epochHasEnded(id)
        marketExists(id)
        returns (uint256 shares)
    {
        if(
            msg.sender != owner &&
-           isApprovedForAll(owner, receiver) == false)
+           isApprovedForAll(owner, msg.sender) == false)
            revert OwnerDidNotAuthorize(msg.sender, owner);
```

