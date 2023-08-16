## Tags

- bug
- help wanted
- 0 (Non-critical)
- disagree with severity
- resolved
- sponsor confirmed

# [Ordering importance in a struct](https://github.com/code-423n4/2022-01-insure-findings/issues/334) 

# Handle

Kumpirmafyas


# Vulnerability details

## Impact
The order of the "struct Template" in the Factory.sol contract is as follows:
1-bool isOpen
2-bool approval
3-bool allowDuplicate
https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L44-L48


The struct above is used in functions as value, in the "key=>value" part in this mapping.
https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L49

When using "template" mapping in this function, it is not done in the defined order, Detail:
- isOpen bool , defined in Struct in the 1st row,
-isOpen bool ,defined in the 1st position in Mapping, naturally
-isOpen bool is defined in the 2nd row in the "approveTemplate" function below.
-The same applies to the approvel bool struct.

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Factory.sol#L101-L103


The problem here is; The order in which Structs are used in a Function is not.
Problem ; The order of the structs in the "key => value" mapping definition affects the function.
Sequencing is important in struct definition in mappings.


## Recommended Mitigation Steps
The order in the struct = the order in the mapping = the order in the function must be the same.

Here ; Sorting in Mapping with Struct is a mandatory condition, while sorting in a function is within the scope of clean code.

