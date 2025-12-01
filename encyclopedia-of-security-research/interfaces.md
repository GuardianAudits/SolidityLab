# Interfaces

Almost every project has interfaces, there are some sneaky issues that can arise from mal-defined interfaces which create issues in production.

## Edge Cases & Exploit Vectors

Whenever an interface does not align with the implementation of the contract being cast as such, this often results in a DoS revert or extreme logical issue.

Most notably, return values are often mis-matching. Whenever an interface expects a return value and none is provided the EVM will revert. Furthermore if a return value of one type is expected, but a different type is received there can be either a DoS or Critical logical error that occurs during decoding the returnData.

## Checklist Items

- Did you check if the interfaces being used for addresses that receive external calls match the actual implementation of that address's implementation of the target function? Function name, parameters, return values.

## Audit References & Resources

MIMSwap: [H-05](https://github.com/GuardianAudits/Audits/blob/main/MIMSwap/2024-03-21_MIMSwap.pdf)

Nunchi SY Token: [H-02](https://github.com/GuardianAudits/Audits/blob/main/Nunchi/2025-11-22_Nunchi_SY_Genesis_Vaults.pdf)

