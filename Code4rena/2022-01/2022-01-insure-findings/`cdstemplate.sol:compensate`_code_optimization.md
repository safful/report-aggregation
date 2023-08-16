## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`CDSTemplate.sol:compensate` code optimization](https://github.com/code-423n4/2022-01-insure-findings/issues/79) 

# Handle

Dravee


# Vulnerability details

## Impact
Duplicated code, loss of maintainability, increased contract size which leads to increased gas cost

## Proof of Concept
The following can be simplified:
```
260:         if (_available >= _amount) {
261:             _compensated = _amount;
262:             _attributionLoss = vault.transferValue(_amount, msg.sender);
263:             emit Compensated(msg.sender, _amount);
264:         } else {
265:             //when CDS cannot afford, pay as much as possible
266:             _compensated = _available;
267:             _attributionLoss = vault.transferValue(_available, msg.sender);
268:             emit Compensated(msg.sender, _available);
269:         }
```
to
```
260:         _compensated = _available >= _amount ? _amount : _available; //when CDS cannot afford, pay as much as possible
261:         _attributionLoss = vault.transferValue(_compensated, msg.sender);
262:         emit Compensated(msg.sender, _compensated);
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Apply the refacto and look out for duplicated code

