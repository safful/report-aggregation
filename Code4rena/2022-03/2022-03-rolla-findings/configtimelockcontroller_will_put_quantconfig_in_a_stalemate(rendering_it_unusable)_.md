## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [ConfigTimeLockController will put QuantConfig in a stalemate(rendering it unusable) ](https://github.com/code-423n4/2022-03-rolla-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/timelock/ConfigTimelockController.sol#L28
https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/QuantConfig.sol#L27


# Vulnerability details

The QuantConfig contract has these important setters, setProtocolAddress(), setProtocolUint256, setProtocolBoolean() and setProtocolRole(). This contract is subjected to a timelock before all such processes above are executed. But, the issue arises in the fact that in configTimeLockController, the state variable minimum delay can be set to an arbitrary value, up to type(uint256).max(cannot assume what value will be set) and could potentially render the QuantConfig contract unusable . All the previous values and addresses would not be able to be changed because of a very high delay being set:

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/timelock/ConfigTimelockController.sol#L28

 I discussed with one of the devs about the use of this specific mapping :

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/QuantConfig.sol#L27

After discussions with one of the devs(#0xca11.eth) , it was understood  that these values are for the rollaOrderFee which is a part of their limit order protocol contract(outside of the scope of the contest) but given the argument above,  its configuration will be severely impacted (old percentage fees won't be able to be changed).Rolla limit order protocol depends on this configuration setting within QuantConfig.

It is recommended that a constant be declared with a MAXIMUM_DELAY and whatever ‘minimum delay’ that is set thereafter should be below this value since there's another function setDelay () which can also be of high arbitrary value:

require(minimum delay ≤MAXIMUM_DELAY, “ too high”)

