## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- edited-by-warden
- responded

# [Beaming job might freeze on dest chain under some conditions, leading to owner loosing (temporarily) access to token](https://github.com/code-423n4/2022-10-holograph-findings/issues/170) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/HolographOperator.sol#L255


# Vulnerability details

## Impact

If the following conditions have been met:
* The selected operator doesn't complete the job, either intentionally (they're sacrificing their bonded amount to harm the token owner) or innocently (hardware failure that caused a loss of access to the wallet) 
* Gas price has spiked, and isn't going down than the `gasPrice` set by the user in the bridge out request

Then the bridging request wouldn't complete and the token owner would loos access to the token till the gas price goes back down again.


## Proof of Concept
The fact that no one but the selected operator can execute the job in case of a gas spike has been proven by the test ['Should fail if there has been a gas spike'](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/test/14_holograph_operator_tests.ts#L834-L844) provided by the sponsor.

An example of a price spike can be in the recent month in the Ethereum Mainnet where the min gas price was 3 at Oct 8, but jumped to 14 the day after and didn't go down since then (the min on Oct 9 was lower than the avg of Oct8, but users might witness a momentarily low gas price and try to hope on it). See the [gas price chat on Etherscan](https://etherscan.io/chart/gasprice) for more details.

## Recommended Mitigation Steps

In case of a gas price spike, instead of refusing to let other operators to execute the job, let them execute the job without slashing the selected operator. This way, after a while also the owner can execute the job and pay the gas price.