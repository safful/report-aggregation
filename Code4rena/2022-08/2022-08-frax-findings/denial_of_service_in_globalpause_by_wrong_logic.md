## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Denial of service in globalPause by wrong logic](https://github.com/code-423n4/2022-08-frax-findings/issues/76) 

# Lines of code

https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairDeployer.sol#L405


# Vulnerability details

## Impact
The method `globalPause` is not tested and it doesn't work as expected.

## Proof of Concept
Because the method returns an array (`_updatedAddresses`) and has never been initialized, when you want to set its value, it fails.

Recipe:

- Call `globalPause` with any valid address.
- The transaction will FAULT.

## Affected source code

- [FraxlendPairDeployer.sol#L405](https://github.com/code-423n4/2022-08-frax/blob/c4189a3a98b38c8c962c5ea72f1a322fbc2ae45f/src/contracts/FraxlendPairDeployer.sol#L405)

## Recommended Mitigation Steps

Initialize the `_updatedAddresses` array like shown bellow:

```diff
    function globalPause(address[] memory _addresses) external returns (address[] memory _updatedAddresses) {
        require(msg.sender == CIRCUIT_BREAKER_ADDRESS, "Circuit Breaker only");
        address _pairAddress;
        uint256 _lengthOfArray = _addresses.length;
+       _updatedAddresses = new address[](_lengthOfArray);
        for (uint256 i = 0; i < _lengthOfArray; ) {
            _pairAddress = _addresses[i];
            try IFraxlendPair(_pairAddress).pause() {
                _updatedAddresses[i] = _addresses[i];
            } catch {}
            unchecked {
                i = i + 1;
            }
        }
    }
```