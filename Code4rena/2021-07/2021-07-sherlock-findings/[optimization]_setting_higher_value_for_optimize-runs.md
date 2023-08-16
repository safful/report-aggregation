## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] Setting higher value for optimize-runs](https://github.com/code-423n4/2021-07-sherlock-findings/issues/83) 

# Handle

hrkrshnn


# Vulnerability details

## Higher value of optimize-runs

``` diff
modified   hardhat.config.js
@@ -25,7 +25,7 @@ module.exports = {
     settings: {
       optimizer: {
         enabled: true,
-        runs: 200,
+        runs: 20000,
       },
     },
   },
```

This value is a tuning parameter for deploy v/s runtime costs. Higher
values optimize for lower runtime cost, which is what you are looking
for. The above value is an example, please decide a suitable high value, and run tests.

