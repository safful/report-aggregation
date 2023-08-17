## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Able to mint any amount of PT](https://github.com/code-423n4/2022-06-illuminate-findings/issues/349) 

# Lines of code

[Lender.sol#L192-L235](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L192-L235)
[Lender.sol#L486-L534](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L486-L534)
[Lender.sol#L545-L589](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L545-L589)


# Vulnerability details

## Impact

Some of the ```lend``` functions do not validate addresses sent as input which could lead to a malicous user being able to mint more PT tokens than they should.

Functions affect:

- [Illuminate and Yield ```lend``` function](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L192-L235).

- [Sense ```lend``` function](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L486-L534).

- [APWine ```lend``` function](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L545-L589).

## Proof of Concept

In the Illuminate and Yield ```lend``` function:

1. Let the Yieldspace pool ```y``` be a malicious contract that implements the ```IYield``` interface.

2. The ```base``` and ```maturity``` functions for ```y``` may return any value so the conditions on lines 208 and 210 are easily passed.

3. The caller of ```lend``` sends any amount ```a``` for the desired underlying ```u```.

4. If principal token ```p``` corresponds to the Yield principal, then the ```yield``` function is called which has a [return value controlled by the malicious contract ```y```](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L648).

5. The ```mint``` function is then called for the principal token with an underlying ```u``` and a maturity ```m``` which will then mint the ```returned``` amount of principal tokens to the malicious user.



In the Sense ```lend``` function:

1. Let the amm ```x``` input variable be a malicous contract that implements the ```ISense``` interface.

2. The malicious user sends any amount of underlying to ```Lender.sol```.

3. Since the amm isn't validated, the ```swapUnderlyingForPTs``` function can return any amount for ```returned``` that is used to mint the Illuminate tokens.

4. The malicious user gains a disproportionate amount of PT.



In the APWine ```lend``` function:

1. Let the APWine ```pool``` input variable be a malicous contract that implements the ```IAPWineRouter``` interface.

2. The malicious user sends any amount of underlying to ```Lender.sol```.

3. The ```swapExactAmountIn``` function of the malicious ```pool``` contract returns any amount for ```returned```.

4. The ```mint``` function is called for the PT with underlying ```u``` and maturity ```m``` with the attacker controlled ```returned``` amount.

## Recommmended Mitigation Steps

Consider validating the input addresses of [```y```](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L197), [```x```](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L492) and [```pool```](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L551) through a whitelisting procedure if possible or validating that the ```returned``` amounts correspond with the amount of PT gained from the protocols by checking the balance before and after the PTs are gained and checking the difference is equal to ```returned```.

