## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [NPM Dependency confusion. Unclaimed NPM Package and Scope/Org](https://github.com/code-423n4/2022-03-timeswap-findings/issues/9) 

# Lines of code

https://github.com/code-423n4/2022-03-timeswap/blob/00317d9a8319715a8e28361901ab14fe50d06172/Timeswap/Convenience/package.json#L40


# Vulnerability details

## Impact
I discovered an npm package and the scope of the package is unclaimed on the NPM website. This will give any User to claim that package and be able to Upload a Malicious Code under that unclaimed package. This results in achieving the Remote code execution on developers/users' machine who depends on the timeswap repository to build it on local env.

##Vulnerable Package Name: @timeswap-labs/timeswap-v1-core

## Proof of Concept
1. Create an Organization called "timeswap-labs".
2. Create a package called "@timeswap-labs/timeswap-v1-core" under "timeswap-labs" Organization.
3. Attacker can able to upload malicious code on unclaimed npm package with a higher version like 99.99.99
4. Now If any user/timeswap developer installs it by npm install package.json. The malicious pkg will be executed.

Till now "The Package is not claimed on NPM Registry, but it's vulnerable to dependency confusion". 
You can read more dependency confusion here: https://dhiyaneshgeek.github.io/web/security/2021/09/04/dependency-confusion/

## Tools Used
Nothing Just OSINT

## Recommended Mitigation Steps
Claim the Scope name called "timeswap-labs" By Following the above POC Step 1.

