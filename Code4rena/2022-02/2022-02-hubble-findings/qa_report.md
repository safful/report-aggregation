## Tags

- bug
- QA (Quality Assurance)
- disagree with severity
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-hubble-findings/issues/124) 

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/AMM.sol#L540

the comment:
```
            makerNotional = newNotional * makerPos / totalPos //<-------- This line
            if (side remains same)
            reducedOpenNotional = takerOpenNotional * makerPos / takerPos
            pnl = makerNotional - reducedOpenNotional

```

and the actual code was 

```
 uint totalPosition = abs(makerPosition + takerPosition).toUint256();
        if (abs(takerPosition) > abs(makerPosition)) {  // taker position side remains same
            uint reducedOpenNotional = takerOpenNotional * abs(makerPosition).toUint256() / 
            abs(takerPosition).toUint256(); 
            uint makerNotional = newNotional * abs(makerPosition).toUint256() / totalPosition; //<------- this line
            pnlToBeRealized = _getPnlToBeRealized(takerPosition, makerNotional, reducedOpenNotional);
```

the line
```
 uint makerNotional = newNotional * abs(makerPosition).toUint256() / totalPosition;
```
was intended to executed outside of `if()` body(Not sure which one is the correct, the comment or the code)