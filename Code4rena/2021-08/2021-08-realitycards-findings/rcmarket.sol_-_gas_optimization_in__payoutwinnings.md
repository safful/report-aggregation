## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Resolved

# [RCMarket.sol - Gas optimization in _payoutWinnings](https://github.com/code-423n4/2021-08-realitycards-findings/issues/6) 

# Handle

PierrickGT


# Vulnerability details

## Impact

We can avoid 3 sload by storing `card[winningOutcome]` in a private variable.

We can also avoid 4 sload by storing `msgSender()` in a private variable.

We can also simplify the `_winningsToTransfer` calculation.

## Proof of Concept

`card[winningOutcome]`:

- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L564
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L574
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L585
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L589

`msgSender()`:

- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L564
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L578
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L585
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L591
- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L592

`_winningsToTransfer`:

- https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L587-L589

## Tools Used

Manual analysis

## Recommended Mitigation Steps

`card[winningOutcome]`:

```
Card storage _cardWinningOutcome = card[winningOutcome];
```

[L574](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L574): `_cardWinningOutcome.rentCollectedPerCard) *`

`msgSender()`:

```
address _msgSender = msgSender();
```

[L578](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L578): `(rentCollectedPerUserPerCard[_msgSender][winningOutcome] *`

[L591 to L592](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L591-L592):

```
_payout(_msgSender, _winningsToTransfer);
emit LogWinningsPaid(_msgSender, _winningsToTransfer);
```

`card[winningOutcome]` and `msgSender()`:

[L564](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L564): `if (_cardWinningOutcome.longestOwner == _msgSender && winnerCut > 0) {`

[L585](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L585): `uint256 _winnersTimeHeld = _cardWinningOutcome.timeHeld[_msgSender];`

`card[winningOutcome]` and `_winningsToTransfer`:

[L587 to L589](https://github.com/code-423n4/2021-08-realitycards/blob/514a6157fb7bd0df1d27a4affece131ba5056818/contracts/RCMarket.sol#L587-L589): `_winningsToTransfer += (_numerator / _cardWinningOutcome.totalTimeHeld);`


