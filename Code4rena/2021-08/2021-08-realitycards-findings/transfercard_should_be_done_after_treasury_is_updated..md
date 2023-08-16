## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [transferCard should be done after treasury is updated.](https://github.com/code-423n4/2021-08-realitycards-findings/issues/35) 

# Handle

0xImpostor


# Vulnerability details

## Impact

When the current owner of the card is still the new owner of the card, `transferCard` is called before the treasury is updated. While this does not currently pose a risk, it is not aligned with best practices of [check-effect-interations](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html) and opens your code to a potential re-entrancy attack in the future.

## Tools Used

Manual analysis

## Recommended Mitigation Steps

```jsx
// line 381
treasury.updateRentalRate(
    _oldOwner,
    _user,
    user[_oldOwner][index[_oldOwner][_market][_card]].price,
    _price,
    block.timestamp
);
transferCard(_market, _card, _oldOwner, _user, _price);
...
// line 449
treasury.updateRentalRate(
    _user,
    _user,
    _price,
    _currUser.price,
    block.timestamp
);
transferCard(_market, _card, _user, _user, _currUser.price);
```

