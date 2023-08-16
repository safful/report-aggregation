## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H0] In the case of Single Asset Entry, new liquidity providers will suffer fund loss due to wrong formula of ΔRo](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/144) 

# Handle

WatchPug


# Vulnerability details

### Current Implementation

#### When `baseToken` rebase up

Per the document: https://github.com/ElasticSwap/elasticswap/blob/a90bb67e2817d892b517da6c1ba6fae5303e9867/ElasticSwapMath.md#:~:text=When%20there%20is%20alphaDecay

and related code: https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L227-L283

`Gamma` is the ratio of shares received by the new liquidity provider when `addLiquidity()` (ΔRo) to the new totalSupply (total shares = Ro' = Ro + ΔRo).

```
ΔRo = (Ro/(1 - γ)) * γ

        Ro * Gamma
    = --------------
         1 - Gamma
⟺
ΔRo * ( 1 - Gamma ) = Gamma * Ro
ΔRo - Gamma * ΔRo = Gamma * Ro
ΔRo = Gamma * Ro + Gamma * ΔRo
           ΔRo    
Gamma = ---------
         Ro + ΔRo 
```

In the current implementation:

```
γ = ΔY / Y' / 2 * ( ΔX / α^ )
```

ΔY is the `quoteToken` added by the new liquidity provider. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L277

Y' is the new Y after `addLiquidity()`, `Y' = Y + ΔY`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L272
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L278

ΔX is `ΔY * Omega`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L259-L263
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L279

α^ is `Alpha - X`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L234-L235
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L280


For instance:

Given:
-   Original State: X = Alpha = 1, Y = Beta = 1, Omega = X/Y = 1
-   When `baseToken` rebase up: Alpha becomes 10
-   Current State: Alpha = 10, X = 1, Y = Beta = 1, Omega = 1


When: new liquidity provider `addLiquidity()` with 4 quoteToken:

```
             4          4 * Omega      16
Gamma = ------------ * ------------ = ----
         (1+4) * 2       10 - 1        90
```
After `addLiquidity()`:

-   baseToken belongs to the newLP: 10 * 16 / 90 = 160 / 90 = 1.7777777777777777
-   quoteToken belongs to the newLP: (1+4) * 16 / 90 = 80 / 90 = 0.8888888888888888
-   In the terms of `quoteToken`, the total value is: 160 / 90 / Omega + 80 / 90 = 240 / 90 = 2.6666666666666665


As a result, the new liquidity provider suffers a fund loss of `4 - 240 / 90 = 1.3333333333333333 in the terms of quoteToken`

The case above can be reproduced by changing the numbers in [this test unit](https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/test/exchangeTest.js#L1804).


#### When `baseToken` rebase down


Per the document: https://github.com/ElasticSwap/elasticswap/blob/a90bb67e2817d892b517da6c1ba6fae5303e9867/ElasticSwapMath.md#:~:text=When%20there%20is%20betaDecay

and related code: https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L297-L363

`Gamma` is the ratio of shares received by the new liquidity provider when `addLiquidity()` (ΔRo) to the new totalSupply (total shares = Ro' = Ro + ΔRo).


```
ΔRo = (Ro/(1 - γ)) * γ

        Ro * Gamma
    = --------------
         1 - Gamma
⟺
ΔRo * ( 1 - Gamma ) = Gamma * Ro
ΔRo - Gamma * ΔRo = Gamma * Ro
ΔRo = Gamma * Ro + Gamma * ΔRo
           ΔRo    
Gamma = ---------
         Ro + ΔRo 
```

In the current implementation:

```
γ = ΔX / X / 2 * ( ΔXByQuoteTokenAmount / β^ )
```

ΔX is the amount of `baseToken` added by the new liquidity provider. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L357

X is the balanceOf `baseToken`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L358

ΔXByQuoteTokenAmount is ΔX / Omega, the value of ΔX in the terms of `quoteToken`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L318-L322
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L329-L333
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L359

β^ is maxΔX / Omega, the value of maxΔX in the terms of `quoteToken`. `maxΔX = X - Alpha`. See:
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L304-L305
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L318-L322
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L341-L342
-   https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L360



For instance:

Given:
-   Original State: X = Alpha = 10, Y = Beta = 10, Omega = X/Y = 1
-   When `baseToken` rebase down, Alpha becomes 1
-   Current State: Alpha = 1, X = 10, Y = Beta = 10, Omega = 1


When: new liquidity provider `addLiquidity()` with `4 baseToken`
```
            4          4 / Omega       8
Gamma = -------- * ---------------- = ----
          10 * 2    (10-1) / Omega     90
```
After `addLiquidity()`:
-   baseToken belongs to the newLP: (1 + 4) * 8 / 90 = 40 / 90 = 0.4444444444444444
-   quoteToken belongs to the newLP: 10 * 8 / 90 = 80 / 90 = 0.8888888888888888
-   In the terms of quoteToken, the total value is: 40 / 90 + 80 / 90 * Omega = 120 / 90 = 1.3333333333333333 < 4

As a result, the new liquidity provider suffers a fund loss of `4 - 120 / 90 = 2.6666666666666665 in the terms of quoteToken`

The case above can be reproduced by changing the numbers in [this test unit](https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/test/exchangeTest.js#L2146).


### The correct formula for ΔRo

#### When baseToken rebase up

```md
When: new liquidity provider addLiquidity with ΔY quoteToken (ΔY <= maxΔY or ΔY <= α^ / ω)

After addLiquidity():
-   baseToken belongs to the newLP: ΔXOfNewLP
-   quoteToken belongs to the newLP: ΔYOfNewLP

ΔY can be divided into 2 parts:
-   ΔYToX: the part used for swap ΔXOfNewLP.  ΔXOfNewLP = ΔYToX * Omega                  (a1)
-   ΔY - ΔYToX: the rest as ΔYOfNewLP

The ratio of newly minted LP tokens for new liquidity provider to the new totalSupply (Ro'): Gamma 

           ΔRo       ΔY - ΔYToX     ΔXOfNewLP
       = --------- = ----------- = -----------                                    (a2)
         Ro + ΔRo     Y + ΔY          Alpha

                     ΔY - ΔYToX     ΔYToX * Omega  
                   = ----------- = ---------------   // substituting (a1)             (a_exp1)
                      Y + ΔY          Alpha        

⟺

(ΔY - ΔYToX) * Alpha = ΔYToX * Omega * (Y + ΔY)
ΔY * Alpha - ΔYToX * Alpha = ΔYToX * Omega * (Y + ΔY)
ΔY * Alpha = ΔYToX * Alpha + ΔYToX * Omega * (Y + ΔY)
           = ΔYToX * ( Alpha + Omega * (Y + ΔY))
               ΔY * Alpha                  ΔY * Alpha            
ΔYToX = ---------------------------  = --------------------                       (a_r1)
         Alpha + Omega * (Y + ΔY)       Alpha + Omega * Y'   


Continue from (a_exp1):

          ΔYToX * Omega 
Gamma  = ---------------
            Alpha       

              ΔY * Omega
       = -----------------------   // substituting (a_r1)                              (a_r2(1))
           Alpha + Omega * Y'

                 ΔY 
       = ---------------------                                                   (a_r2(2)) 
           Alpha/Omega + Y'


Gamma is the ratio of ΔY to the total amounts of baseToken and quoteToken after addLiquidity:
-   (a_r2(1)) is the formula in the terms of baseToken
-   (a_r2(2)) is the formula in the terms of quoteToken



Based on (a2):

           ΔRo    
Gamma = ---------
         Ro + ΔRo 
⟺
ΔRo = Gamma * Ro + Gamma * ΔRo
ΔRo - Gamma * ΔRo = Gamma * Ro
ΔRo * ( 1 - Gamma ) = Gamma * Ro
        Ro * Gamma
ΔRo = --------------
         1 - Gamma

```

#### When baseToken rebase down

```md
When: new liquidity provider addLiquidity with ΔX baseToken (ΔX <= maxΔX or ΔY <= α^)

After addLiquidity()
-   baseToken belongs to the newLP: ΔXOfNewLP
-   quoteToken belongs to the newLP: ΔYOfNewLP

ΔX can be divided into 2 parts:
-   ΔXToY:  the part used for swap ΔYOfNewLP. ΔYOfNewLP = ΔXToY / Omega                 (b1)
-   ΔX - ΔXToY: the rest as ΔXOfNewLP


The ratio of newly minted LP tokens for new liquidity provider to the new totalSupply (Ro'): Gamma

          ΔRo        ΔX - ΔXToY     ΔYOfNewLP   
      = --------- = ------------ = ------------                                  (b2)
        Ro + ΔRo     Alpha + ΔX         Y     

                                 ΔXToY / Omega
                               = -------------    // substituting (b1)
                                      Y

                                     ΔXToY     
                               = -------------                                   (b_exp1)
                                   Y * Omega   

⟺

(ΔX - ΔXToY) * Y = (Alpha + ΔX) * ΔXToY / Omega
ΔX * Y - ΔXToY * Y = (Alpha + ΔX) * ΔXToY / Omega
ΔX * Y = ΔXToY * Y + (Alpha + ΔX) * ΔXToY / Omega
       = ΔXToY * ( Y + (Alpha + ΔX) / Omega )

                  ΔX * Y
ΔXToY = --------------------------                                               (b_r1)
         Y + (Alpha + ΔX) / Omega



Continue from (b_exp1)

            ΔXToY    
Gamma = -------------
          Y * Omega  

                  ΔX * Y
      = ----------------------------------------    // substituting (b_r1)
         (Y + (Alpha + ΔX) / Omega) * Y * Omega

               ΔX / Omega
      = ---------------------------                                               (b_r2(2))
         Y + (Alpha + ΔX) / Omega

                   ΔX
      = ---------------------------                                               (b_r2(1))
         Y * Omega + (Alpha + ΔX)

               ΔX
      = -------------------  // substituting (Omega = X/Y)
         X + (Alpha + ΔX)



Gamma is the ratio of ΔX to the total amounts of baseToken and quoteToken after addLiquidity:
-   (b_r2(1)) is the formula in the terms of baseToken
-   (b_r2(2)) is the formula in the terms of quoteToken


Based on (b2):

           ΔRo    
Gamma = ---------
         Ro + ΔRo 
⟺
        Ro * Gamma
ΔRo = --------------
         1 - Gamma

```

### Recommendation

Update code and document using the correct formula for ΔRo.

