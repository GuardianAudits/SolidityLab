# ✋ Contract Cannot Accept Ether DoS

This article explores a specific DoS vulnerability known as "Contract Cannot Accept Ether". <br>
We'll delve into how attackers exploit this weakness and explore strategies to fortify your smart contracts against such attacks.

## What is DOS ?

Denial-of-Service (DoS) attacks aim to disrupt the normal operation of a contract, preventing legitimate users from interacting with it. <br>
Smart contracts, while powerful tools for decentralized applications, can be susceptible to malicious attacks. <br>

## Example

Let's use the famous example from the Ethernaut's CTF.<br>

The Goal of this contract is to be and stay the King.<br>
It's just an auction contract, whoever pays more than the previous bid get to be the new King.<br>
The old king will then receive the money sent by the new King


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King 👑 {

  address king;
  uint public prize;
  address public owner;

  constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    // Will fail if the King is a Smart Contract and can't receive Ether
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }
}
```

By default, smart contracts don't accept Ether.<br>
For accepting them, there's a need for a `receive` and/or `fallback` function.<br>
The problem with this contract is the fact  it assume that the new king is a EOA (i.e. a Wallet) and can accept Ether.
However is the next example we can see that the caller isn't a Wallet but instead a smart contract that can't accept Ether.
It will revert if the user's contract look like this :

```solidity
contract CannotAcceptETH{
  constructor(address _kingContract) payable {
    payable(_kingContract).call{value: msg.value}("");
  }
}
```
Indeed, the above contract has no `receive` nor `fallback` function.


From there the contract is Doom as soon as a contract that didn't implement those function becomes the new King.<br>
The King will stay King forever but could never get back the money they bid.<br>

## Mitigation and Best Practice:

In this example to prevent this we could : <br><br>

  - Give access to the `owner` to remove the current king (but it would not be decentralized).
```solidity
    // Emergency function to reclaim the throne if Ether transfer fails persistently
    function reclaimThrone() external {
        require(msg.sender == owner, "Only the owner can reclaim the throne");
        king = owner;
    }
```
  - Only accept ERC-20 like WETH, that will not trigger a revert.

This kind of Attack through DOS is one of many ways things can go wrong.
A good rule of thumb while auditing is to be extra careful when calling an arbitrary address.

Conclusion:

This was an example of a User contract that can't accept Ether and DOS the King contract.

But it's quite frequent to see protocols that forgot to set up a `receive` and/or `fallback`, so no one can send Ether to the contract ever.