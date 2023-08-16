## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Not needed zero address check](https://github.com/code-423n4/2021-12-sublime-findings/issues/160) 

# Handle

0x0x0x


# Vulnerability details

[https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Verification/Verification.sol#L150](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Verification/Verification.sol#L150)

In `Verfication.sol#unlinkAddress`, there is a not needed zero address check.

```

require(_linkedTo != address(0), 'V:UA-Address not linked');
require(_linkedTo == msg.sender, 'V:UA-Not linked to sender');

```

Since, `msg.sender != address(0)`, there is no need for a zero address check here.

