## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas in `Bitmap.sol:getMSB()`: unnecessary arithmetic operation](https://github.com/code-423n4/2022-01-notional-findings/issues/128) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost due to unnecessary arithmetic operation

## Proof of Concept
See the @audit-info tag:
```
File: Bitmap.sol
46:     function getMSB(uint256 x) internal pure returns (uint256 msb) {
47:         // If x == 0 then there is no MSB and this method will return zero. That would
48:         // be the same as the return value when x == 1 (MSB is zero indexed), so instead
49:         // we have this require here to ensure that the values don't get mixed up.
50:         require(x != 0); // dev: get msb zero value
51:         if (x >= 0x100000000000000000000000000000000) {
52:             x >>= 128;
53:             msb += 128; //@audit-info this one and only can be replaced with = 
54:         }
55:         if (x >= 0x10000000000000000) {
56:             x >>= 64;
57:             msb += 64;
58:         }
59:         if (x >= 0x100000000) {
60:             x >>= 32;
61:             msb += 32;
62:         }
63:         if (x >= 0x10000) {
64:             x >>= 16;
65:             msb += 16;
66:         }
67:         if (x >= 0x100) {
68:             x >>= 8;
69:             msb += 8;
70:         }
71:         if (x >= 0x10) {
72:             x >>= 4;
73:             msb += 4;
74:         }
75:         if (x >= 0x4) {
76:             x >>= 2;
77:             msb += 2;
78:         }
79:         if (x >= 0x2) msb += 1; // No need to shift xc anymore
80:     }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
On line 53, and only there, it is absolutely certain that `+=` can be replaced with `=`, which would look like this:
```
53:             msb = 128;
```

