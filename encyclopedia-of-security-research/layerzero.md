# LayerZero


# LayerZero Messaging

## Edge Cases & Exploit Vectors

- When sending requests through the endpoint send function, the refund receiver will receive Ether refunds. This address could re-enter, gas grief, DoS etc...
- Many production tokens (e.g., GMX) do not return a boolean from their mint/burn methods. Calling them through LayerZero’s standard [IMintableBurnable](https://github.com/LayerZero-Labs/devtools/blob/aa43d67ec0ca5a1668b9174a3f3a032e6ce8693d/packages/oft-evm/contracts/interfaces/IMintableBurnable.sol), which expects a bool, will revert. For such tokens use an interface without a return value, e.g.:
```solidity
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.22;

    interface IGMXMinterBurnable {
        function burn(address _account, uint256 _amount) external;
        function mint(address _account, uint256 _amount) external;
    }
```

## Checklist Items

- Did you check the refundReceiver specified in the send call for DoS, Re-entrancy, Gas griefing?
- Did you verify the mint/burn interface matches the underlying token (i.e., it may not return a bool)?


## Audit References & Resources



# lzRead

## Edge Cases & Exploit Vectors

- If the target lzRead function reverts instead of successfully executing then the messaging channel can become stuck. [Reference](https://x.com/GuardianAudits/status/1934234468841316601)
- Re-orgs can cause a mis-reported owner at the target block if the confirmations is not set high enough
- The delay between what block/timestamp an lzRead function is queried at and when the lzReceive result comes back is a dangerous no-man's land period. When an lzRead result is reported back through lzReceive, know that the result only attests to the state of the target contract/chain a handful of seconds ago, and that state could have changed.
- With smart contract wallets/multisig wallets it's possible that one user owns address 0xA on one chain and a different user owns the same address 0xA on a different chain 
- If the gas specified in the options is insufficient for executing the lzReceive invocation upon receiving the lzRead result then it will not be executed automatically and must be manually invoked through the LzEndpoint
- The returnDataSize specified in the options must match exactly the size of bytes returned from the lzRead function always
- Once the lzRead result has been verified by all DVNs then anyone can execute it through the LzEndpoint. A malicious actor could frontrun the lzExecutor and invoke this message with insufficient gas, or while they've put the protocol in some invalid state
- When sending lzRead requests through the endpoint send function, the refund receiver will receive Ether refunds. This address could re-enter, gas grief, DoS etc...
- Be sure to take ample time to think about state across multiple chains and across time, and how actions being taken on any chain at any point in time can lead to a potential invalid state -- especially if state on one chain has not been synced to that of others.

## Checklist Items

- Did you fuzz the target read function to ensure it never reverts?
- Did you check that the confirmations is configured high enough for each individual target chain that can be used?
- Did you consider different owners of the same address across chains?
- Did you check that the gas specified in the options is sufficient to execute lzReceive with the returned lzRead result in all cases?
- Did you check that the returnDataSize specified in the options matches exactly the size of bytes returned in all cases?
- Did you verify that the correct lzRead channel is being used?
- lzReceive must only be callable by the endpoint
- Did you consider malicious executors of the lzRead result message through the lz endpoint? E.g. for insufficinet gas during execution (censorying) or execution while in an invalid state?
- Did you check the refundReceiver specified in the send call for DoS, Re-entrancy, Gas griefing?
- If users are allowed to supply their own options, enforcedOptions should always be combined with the user supplied options

## Audit References & Resources

- [Yuga Labs Shadows 1](https://github.com/GuardianAudits/Audits/blob/main/YugaLabs/2025-01-17_YugaLabs_NFT_Shadows.pdf)
- [Yuga Labs Shadows 2](https://github.com/GuardianAudits/Audits/blob/main/YugaLabs/2025-02-05_YugaLabs_NFT_Shadows_2.pdf)
- [Azuki Animecoin](https://github.com/GuardianAudits/Audits/tree/main/Animecoin)
- [lzRead Blocked Channel Thread](https://x.com/GuardianAudits/status/1934234468841316601)
