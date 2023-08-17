## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report
- responded

# [An attacker can manipulate each pod and gain an advantage over the remainder Operators](https://github.com/code-423n4/2022-10-holograph-findings/issues/168) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L484-L539
https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L1138-L1144


# Vulnerability details

# H001 An attacker can manipulate each pod and gain an advantage over the remainder Operators

## Impact

In [contracts/HolographOperator.sol#crossChainMessage](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L484-L539), each Operator is selected by:

- Generating a random number ([L499](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L499))
- A pod is selected by dividing the random with the total number of pods, and using the remainder ([L503](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L503))
- An Operator of the selected pod is chosen using the **same** random and dividing by the total number of operators ([L511](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L511)).

This creates an unintended bias since the first criterion (the `random`) is used for both selecting the pod and selecting the Operator, as explained in a previous issue (`M001-Biased distribution`). In this case, an attacker knowing this flaw can continuously monitor the contracts state and see the current number of pods and Operators. Accordingly to the [documentation](https://docs.holograph.xyz/holograph-protocol/operator-network-specification#operator-job-selection) and provided [flow](https://github.com/code-423n4/2022-10-holograph/blob/main/docs/IMPORTANT_FLOWS.md#joining-pods):

* An Operator can easily join and leave a pod, albeit when leaving a small fee is paid
* An Operator can only join one pod, but an attacker can control multiple Operators
* The attacker can then enter and leave a pod to increase (unfairly) his odds of being selected for a job

Honest Operators may feel compelled to leave the protocol if there are no financial incentives (and lose funds in the process), which can also increase the odds of leaving the end-users at the hands of a malicious Operator.

## Proof of Concept

Consider the following simulation for 10 pods with a varying number of operators follows (X → "does not apply"):
| Pod n | Pon len | Op0 | Op1 | Op2 | Op3 | Op4 | Op5 | Op6 | Op7 | Op8 | Op9 | Total Pod |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | 
| P0 | 10 | 615 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 615 |
| P1 | 3 | 203 | 205 | 207 | X | X | X | X | X | X | X | 615 |
| P2 | 6 | 208 | 0 | 233 | 0 | 207 | 0 | X | X | X | X | 648 |
| P3 | 9 | 61 | 62 | 69 | 70 | 65 | 69 | 61 | 60 | 54 | X | 571 |
| P4 | 4 | 300 | 0 | 292 | 0 | X | X | X | X | X | X | 592 |
| P5 | 10 | 0 | 0 | 0 | 0 | 0 | 586 | 0 | 0 | 0 | 0 | 586 |
| P6 | 2 | 602 | 0 | X | X | X | X | X | X | X | X | 602 |
| P7 | 7 | 93 | 93 | 100 | 99 | 76 | 74 | 78 | X | X | X | 613 |
| P8 | 2 | 586 | 0 | X | X | X | X | X | X | X | X | 586 |
| P9 | 6 | 0 | 190 | 0 | 189 | 0 | 192 | X | X | X | X | 571 |

At this stage, an attacker Mallory joins the protocol and scans the protocol (or interacts with - e.g. `getTotalPods`, `getPodOperatorsLength`). As an example, after considering the potential benefits, she chooses pod `P9` and sets up some bots `[B1, B2, B3]`. The number of Operators will determine the odds, so:

| Pod P9 | Alt len | Op0 | Op1 | Op2 | Op3 | Op4 | Op5 | Op6 | Op7 | Op8 | Op9 | Total Pod |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | 
| P9A | 4 | 0 | 276 | 0 | 295 | X | X | X | X | X | X | 571 |
| P9B | 5 | 0 | 0 | 0 | 0 | 571 | X | X | X | X | X | 571 |
| P9 | 6 | 0 | 190 | 0 | 189 | 0 | 192 | X | X | X | X | 571 |
| P9C | 7 | 66 | 77 | 81 | 83 | 87 | 90 | 87 | X | X | X | 571 |
| P9D | 8 | 0 | 127 | 0 | 147 | 0 | 149 | 0 | 148 | X | X | 571 |

And then:

1. She waits for the next job to fall in `P9` and keeps an eye on the number of pods, since it could change the odds.
2. After an Operator is selected (he [pops](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L518) from the array), the number of available Operators change to 5, and the odds change to `P9B`.
3. She deploys `B1` and it goes to position `Op5`, odds back to `P9`. If the meantime the previously chosen Operator comes back to the `pod`, see the alternative timeline.
4. She now has 1/3 of the probability to be chosen for the next job:
4.1 If she is not chosen, [she will assume the position](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L1138-L1144) of the chosen Operator, and deploys `B2` to maintain the odds of `P9` and controls 2/3 of the pod.
4.2 If she is chosen, she chooses between employing another bot or waiting to execute the job to back to the pod (keeping the original odds).
5. She can then iterate multiple times to swap to the remainder of possible indexes via step 4.1. 

Alternative timeline (from previous 3.):
1. The chosen Operator finishes the job and goes back to the pod. Now there's 7 members with uniform odds (`P9C`).
2. Mallory deploys `B2` and the length grows to 8, the odds turn to `P9D` and she now controls two of the four possible indexes from which she can be chosen.

There are a lot of ramifications and possible outcomes that Mallory can manipulate to increase the odds of being selected in her favor.

## Tools Used

Manual

## Recommended Mitigation Steps

Has stated in `M001-Biased distribution`, use two random numbers for pod and Operator selection. Ideally, an independent source for randomness should be used, but following the assumption that the one used in [L499](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L499) is safe enough, using the most significant bits (e.g. `random >> 128`) should guarantee an unbiased distribution. Also, reading the [EIP-4399](https://eips.ethereum.org/EIPS/eip-4399) could be valuable.

Additionally, since randomness in blockchain is always tricky to achieve without an oracle provider, consider adding additional controls (e.g. waiting times before joining each pod) to increase the difficulty of manipulating the protocol.

And finally, in this particular case, removing the swapping mechanism (moving the last index to the chosen operator's current index) for another mechanism (shifting could also create conflicts [with backup operators?](https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L358-L370)) could also increase the difficulty of manipulating a particular pod.

