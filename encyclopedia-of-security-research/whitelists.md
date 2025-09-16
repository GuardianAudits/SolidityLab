# Whitelists

Many protocols today implement whitelists for regulatory compliance, here are some common pitfalls and vulnerabilities with such patterns.

## Edge Cases & Exploit Vectors

- Any whitelisted address can use EIP 7702 to set their own account code that allows non-whitelsited accounts to interact with the protocol

Consider the following scenario:

- Assume protocol has a whitelist mapping, where only users who are whitelisted can call function A
- Bob is whitelisted for his address 0xFF
- Bob uses 7702 to set his 0xFF account code to:

```solidity
contract {
     address victimSystem;
    
     function callThis(...) external {
           victimSystem.whitelistedFunction(...);
     }
}
```

- Now anyone can call the whitelisted function through calling callThis on 0xFF

## Checklist Items

- Did you check if EIP 7702 could be used to bypass the whitelist?


## Audit References & Resources

- M0 Uniswap V4 hook review, L-07: [Pectra Upgrade Enables EOAs](https://github.com/GuardianAudits/Audits/blob/main/M0/M0_Uniswap_V4_Hooks_report.pdf)
- Bracket Wrapped Vault Review: M-01: [TODO]