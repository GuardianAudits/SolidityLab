# 🤷♂ Lack of Access Control

## What is it even

Access control is the application of constraints on who or what is authorized to perform actions or access resources.
In the world of smart contracts, this means, clearly defining what various users of the system are permitted to do.
Weak access control or lack of can allow attackers to access critical functions that can lead to loss of funds.
<br />
Imagine an auction contract where anyone can set the auction deadline, or an amm which allows anyone to set the fee or worse—anyone can call the `pause()` function and freeze the entire protocol, halting trading or token transfers. See how this such a big deal? <br />

There are various ways of implementing access control. You can learn more from [openzeppelin docs](https://docs.openzeppelin.com/contracts/5.x/access-control)

## 🧨 Real Case: The HospoWise Hack

An interesting case study is, the **HospoWise Hack** where a public `burn()` function allowed anyone to burn tokens.
A lack of access control on a sensitive function led to direct loss of value.

## Internal/private vs public/external functions

This are used to determine the level of accessibility for any function:

- Internal and private function can only be called by the contract
- Public and external functions can be called by the contract but are also exposed to the public ie anyone can call them.
  See [solidity docs](https://docs.soliditylang.org/en/v0.8.30/contracts.html#function-visibility) for more details

**🐛 What Can Go Wrong?**

### [HIGH: Attacker can steal all tokens as a result of the payWithERC20() function being public](https://github.com/sherlock-audit/2025-03-crestal-network-judging/issues/325)

The function `payWithERC20()` on payment.sol is used to facilitate payments when creating agents and when user makes a top up.
The bug happens because this function is exposed to the public and as such can be called by anyone. An attacker can call this function to steal any approved tokens from any address

```solidity
    function payWithERC20(address erc20TokenAddress, uint256 amount, address fromAddress, address toAddress) public {
        // check from and to address
        require(fromAddress != toAddress, "Cannot transfer to self address");
        require(toAddress != address(0), "Invalid to address");
        require(amount > 0, "Amount must be greater than 0");
        IERC20 token = IERC20(erc20TokenAddress);
        token.safeTransferFrom(fromAddress, toAddress, amount);
    }
```

Guess how they fixed this:

Well, just changed the visibility to internal and Voila, code is secure

```solidity
    function payWithERC20(address erc20TokenAddress, uint256 amount, address fromAddress, address toAddress) internal {
    }
```

See how important it is to validate visibility is set correctly.

## [Anyone Can Cancel Market Orders – Missing Ownership Check](https://github.com/solodit/solodit_content/blob/main/reports/Cyfrin/2024-07-13-cyfrin.zaros.md)

The function `cancelMarketOrder` is meant to cancel an active market order. This is a critical function and as such only owner of the order is supposed to call it.

The function is implemented as follows:

```solidity
function cancelMarketOrder(uint128 tradingAccountId) external {
    MarketOrder.Data storage marketOrder = MarketOrder.loadExisting(tradingAccountId);

    marketOrder.clear();

    emit LogCancelMarketOrder(msg.sender, tradingAccountId);
}
```

Note, the implementation fails to check if the caller is the owner of this order. This allows anyone to cancel a market order.

## 🤦‍♂️ Wrong implementation of access control

The following is extracted from [rabbithole contest on c4](https://code4rena.com/audits/2023-01-rabbithole-quest-protocol-contest)
The protocol has a function `mint()` which is used to mint receipts. Minting is however not intended for everyone, only the minter address is allowed to do so.

```solidity
    function mint(address to_, string memory questId_) public onlyMinter {
        _tokenIds.increment();
        uint newTokenID = _tokenIds.current();
        questIdForTokenId[newTokenID] = questId_;
        timestampForTokenId[newTokenID] = block.timestamp;
        _safeMint(to_, newTokenID);
    }
```

They clearly know this function is critical hence the `onlyMinter` restriction.

Let's take a look at the `onlyMinter` modifier

```solidity
    modifier onlyMinter() {
        msg.sender == minterAddress;
        _;
    }
```

Do you see the problem?

The modifier is just checking if **caller is minter but does nothing with the result**. ie it's only saying, please check if this two are the same and proceed to execute the function regardless.

The intention to limit this to only minter is there, but a broken implementation allows anyone to mint receipts.
The fix is easy, we just need to do something with the result of `msg.sender == minterAddress` in this case, **we should revert if msg.sender is not equal to minterAddress**

```solidity
    modifier onlyMinter() {
        require(msg.sender == minterAddress,"OnlyMinter");
        _;
    }
```

Another variation where implementation is flawed: [C4 finding](https://code4rena.com/audits/2025-01-liquid-ron/submissions/S-395)

## Conclusion

Access control is a critical aspect of smart contract security—especially when dealing with user funds. Poor or missing access control can allow attackers to manipulate contract behavior or drain funds entirely. For protocol developers, it's important to recognize that access control vulnerabilities can take many forms, as demonstrated in the examples above. For security researchers, it's equally vital to thoroughly analyze all privileged functions and ensure proper role restrictions are enforced.
**Access control bugs are rarely complex but they’re often expensive.**
