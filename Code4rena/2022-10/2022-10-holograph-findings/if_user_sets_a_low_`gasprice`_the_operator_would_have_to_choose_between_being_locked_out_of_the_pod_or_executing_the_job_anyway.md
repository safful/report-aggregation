## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected for report
- responded

# [If user sets a low `gasPrice` the operator would have to choose between being locked out of the pod or executing the job anyway](https://github.com/code-423n4/2022-10-holograph-findings/issues/364) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/HolographOperator.sol#L202-L340
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L593-L596
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/module/LayerZeroModule.sol#L277-L294


# Vulnerability details

During the beaming process the user compensates the operator for the gas he has to pay by sending some source-chain-native-tokens via `hToken`.
The amount he has to pay is determined according to the `gasPrice` set by the user, which is supposed to be the maximum gas price to be used on dest chain (therefore predicting the max gas fee the operator would pay and paying him the same value in src chain native tokens).
However, in case the user sets a low price (as low as 1 wei) the operator can't skip the job because he's locked out of the pod till he executes the job.
The operator would have to choose between loosing money by paying a higher gas fee than he's compensated for or being locked out of the pod - not able to execute additional jobs or get back his bonded amount.


## Impact
Operator would be loosing money by having to pay gas fee that's higher than the compensation (gas fee can be a few dozens of USD for heavy txs).
This could also be used by attackers to make operators pay for the attackers' expensive gas tasks:
* They can deploy their own contract as the 'source contract'
* Use the `bridgeIn` event and the `data` that's being sent to it to instruct the source contract what operations need to be executed
* They can use it for execute operations where the `tx.origin` doesn't matter (e.g. USDc gasless send)

## Proof of Concept
* An operator can't execute any further jobs or leave the pod till the job is executed. From [the docs](https://docs.holograph.xyz/holograph-protocol/operator-network-specification#:~:text=When%20an%20operator%20is%20selected%20for%20a%20job%2C%20they%20are%20temporarily%20removed%20from%20the%20pod%2C%20until%20they%20complete%20the%20job.%20If%20an%20operator%20successfully%20finalizes%20a%20job%2C%20they%20earn%20a%20reward%20and%20are%20placed%20back%20into%20their%20selected%20pod.):
> When an operator is selected for a job, they are temporarily removed from the pod, until they complete the job. If an operator successfully finalizes a job, they earn a reward and are placed back into their selected pod.
* Operator can't skip a job. Can't prove a negative but that's pretty clear from reading the code.
* There's indeed a third option - that some other operator/user would execute the job instead of the selected operator, but a) the operator would get slashed for that. b) If the compensation is lower than the gas fee then other users have no incentive to execute it as well.

## Recommended Mitigation Steps

Allow operator to opt out of executing the job if the `gasPrice` is higher than the current gas price