## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [RCFactory: Do multiplication instead of division for length checks](https://github.com/code-423n4/2021-08-realitycards-findings/issues/39) 

# Handle

hickuphh3


# Vulnerability details

### Impact

Solidity division rounds down, so doing `M / 2 <= N` checks mean that `M` can be at most `2N + 1`.

This affects the following checks:

```jsx
require(
	(_tokenURIs.length / 2) <= cardLimit,
	"Too many tokens to mint"
);

require(
  _cardAffiliateAddresses.length == 0 ||
	  _cardAffiliateAddresses.length == (_tokenURIs.length / 2),
  "Card Affiliate Length Error"
)
```

Note that with the current implementation, if `_tokenURIs` is of odd length, its last element will be  redundant, but market creation will not revert.

The stricter checks will partially mitigate `_tokenURIs` having odd length because `_cardAffiliateAddresses` is now required to be exactly twice that of `_tokenURIs`.

### Recommended Mitigation Steps

These checks should be modified to 

```jsx
require(
	_tokenURIs.length <= cardLimit * 2,
	"Too many tokens to mint"
);

require(
  _cardAffiliateAddresses.length == 0 ||
	  _cardAffiliateAddresses.length * 2 == _tokenURIs.length,
  "Card Affiliate Length Error"
);
```

In addition, consider adding a check for `_tokenURIs` to strictly be of even length.

`require(_tokenURIs.length % 2 == 0, "TokenURI Length Error");`

