## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Registry.sol works bad - it fails to delivere expected functionality](https://github.com/code-423n4/2022-08-mimo-findings/issues/78) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyFactory.sol#L40-L58
https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyRegistry.sol#L39-L59


# Vulnerability details

## Impact
The description of Registry.sol is following:
/// Deploys new proxies via the factory and keeps a registry of owners to proxies. Owners can only
/// have one proxy at a time.
But it is not.
There are multiple problems:
1) Proxy owner can change and will not be registered
2) There many ways for an owner to have many proxies:
- a few other proxy owners transfeOwnership() to one address
- Registry tracks last deployments and does not guarantee ownership
- Factory.sol allows calling deployFor() to anyone, without any checks and registrations

## Proof of Concept
https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyFactory.sol#L40-L58
https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyRegistry.sol#L39-L59

## Tools Used
Hardhat

## Recommended Mitigation Steps
Delete Proxy.transfetOwnership()
Disallow anyone to call deploy() and deployFor() in Factory()