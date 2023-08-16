## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Checking zero address on msg.sender is impractical](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/103) 

# Handle

dalgarim


# Vulnerability details

## Impact
sYETIToken.sol mint function checks if msg.sender is zero address.
It is extremely unlikely that someone possesses a private key of zero address.
This 'require' statement semantically has no meaning

## Proof of Concept
[mint](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/sYETIToken.sol#L175)
```
function mint(uint256 amount) public returns (bool) {
        require(msg.sender != address(0), "Zero address");
        User memory user = users[msg.sender];

        uint256 shares = totalSupply == 0 ? amount : (amount * totalSupply) / effectiveYetiTokenBalance;
        user.balance += shares.to128();
        user.lockedUntil = (block.timestamp + LOCK_TIME).to128();
        users[msg.sender] = user;
        totalSupply += shares;

        yetiToken.sendToSYETI(msg.sender, amount);
        effectiveYetiTokenBalance = effectiveYetiTokenBalance.add(amount);

        emit Transfer(address(0), msg.sender, shares);
        return true;
    }
```

## Tools Used
Manual

## Recommended Mitigation Steps
The require statement can be removed

