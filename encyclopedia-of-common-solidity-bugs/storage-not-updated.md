# ⬆ Storage Not Updated: Hidden Pitfalls in Smart Contract Logic

The Ethereum Virtual Machine has different areas where it can store data with the most prominent being storage, transient storage, memory and the stack.<br/>
State variables are variables whose values are either permanently stored in contract storage or, alternatively, temporarily stored in transient storage which is cleaned at the end of each transaction

see [Storage in smart contracts](https://docs.soliditylang.org/en/v0.8.30/introduction-to-smart-contracts.html#locations)

Smart contracts often rely on state variables to track things like balances, participants, goals, or limits. If those variables aren't updated when something changes, it can lead to incorrect logic, wasted gas, or even denial of service.

Let's take a look at the following code

```solidity
pragma solidity ^0.8.0;

contract Simple {
    uint256 public total;

    function increase(uint256 amount) external {
        total += amount;
        emit totalIncreased();
    }

    function decrease(uint256 amount) external {
        // Intentionally forgetting to update `total`
        // total -= amount;  <-- missing!
        emit totalReduced();
    }
}
```

In the example above, `decrease()` gives the impression that something has changed, but the state variable total remains untouched. This kind of oversight especially when buried in larger, more complex contracts can break entire protocols.

## Real Example: A Fundraising Contract Gone Wrong

Let’s look at a real world example from a fundraising contract. The contract tracks total contributions using a variable called **totalRaised** and allows users to call `contribute()` to donate funds

Whenever users call `contribute()` the following state updates happen

```solidity
    function contribute() public payable nonReentrant {
        require(!goalReached, "Goal already reached");
        require(block.timestamp < fundraisingDeadline, "Deadline hit");
        require(msg.value > 0, "Contribution must be greater than 0");

        //@<truncated for brevity>

        uint256 effectiveContribution = msg.value;
        if (totalRaised + msg.value > fundraisingGoal) {
            effectiveContribution = fundraisingGoal - totalRaised;
            payable(msg.sender).transfer(msg.value - effectiveContribution);
        }

        if (contributions[msg.sender] == 0) {
            contributors.push(msg.sender);
        }


        contributions[msg.sender] += effectiveContribution;
        totalRaised += effectiveContribution;
```

This updates several important state variables:

- The contributer is added to an array of contributors `contributors.push(msg.sender);`
- We update the senders contribution `contributions[msg.sender] += effectiveContribution;`
- we updated the total Raised amount `totalRaised += effectiveContribution;`

If the `fundraisingGoal` is not reached , the contract allows contributors to get a refund by calling `refund()`

**What do you expect to happen with our state variables?**
Well let's see how `refund()` is implemented:

```solidity
    function refund() external nonReentrant {
        require(!goalReached, "Fundraising goal was reached");
        require(
            block.timestamp > fundraisingDeadline,
            "Deadline not reached yet"
        );
        require(contributions[msg.sender] > 0, "No contributions to refund");

        uint256 contributedAmount = contributions[msg.sender];
        contributions[msg.sender] = 0;

        payable(msg.sender).transfer(contributedAmount);

        emit Refund(msg.sender, contributedAmount);
    }
```

Now, what is refund is supposed to do, basically we are redoing what `contribute()` did or that's what we expect to happen.

But something sus is happening with our code above, only one of the state variables we had previously seems to be referenced here ie `contributions[msg.sender] = 0;`

The `refund()` function correctly resets the caller's contribution but fails to do 2 very important things

1. The `totalRaised` variable is not updated even though we refund the caller.
2. The array of contributers is not updated to reflect that one contributor has exited

Let's explore the first scenario:

### What's the problem

Even if funds are no longer present, the contract behaves as if they are, enabling logic based on faulty assumptions.

The contract has a function that allows the admin to extend the fundraiser if the goal was not reached and the fundraising deadline has passed.

Do you see where this is going.

The contract will think more money was raised than actually exists.

**🧪 Example**

- Fundraising goal = 50K
- User A contributes 20K
- User B contributes 10K

`→ totalRaised = 30K`

Deadline passes, goal not met.

User B calls `refund()` and gets their 10K back.
→ `contributions[B] = 0`, but `totalRaised = 30K` (still!)

If the protocol admin later extends the deadline, and a new user contributes 20K,
→ `totalRaised = 50K` —> goal appears reached!

**But in reality, only 40K is available. The protocol is now working off a false state.**

**The fix?**

```diff
    function refund() external nonReentrant {
        require(!goalReached, "Fundraising goal was reached");
        require(
            block.timestamp > fundraisingDeadline,
            "Deadline not reached yet"
        );
        require(contributions[msg.sender] > 0, "No contributions to refund");
        uint256 contributedAmount = contributions[msg.sender];
        contributions[msg.sender] = 0;
     + totalRaised -= contributedAmount
        payable(msg.sender).transfer(contributedAmount);
        emit Refund(msg.sender, contributedAmount);
    }
```

## 🧠 Storage vs. Memory: Another Gotcha

Well, state variables can come in various forms, think of making changes to a struct in memory instead of storage

Let's see another variation: we have a crowd fund contract.

```solidity
    struct Campaign {
        // Creator of campaign
        address creator;
        // Amount of tokens to raise
        uint256 goal;
        // Total amount pledged
        uint256 pledged;
        // Timestamp of start of campaign
        uint32 startAt;
        // Timestamp of end of campaign
        uint32 endAt;
        // True if goal was reached and creator has claimed the tokens.
        bool claimed;
    }
```

The above struct stores the info about the crowd fund(ongoing campaign)
When launched, the goal is set as well as start and end time.

We also define a mapping to track the total pledged as shown below.

```solidity
    // Mapping from id to Campaign
    mapping(uint256 => Campaign) public campaigns;
    // Mapping from campaign id => pledger => amount pledged
    mapping(uint256 => mapping(address => uint256)) public pledgedAmount;

```

Now, let's take a look at the function users would call to pledge an amount.

```solidity

    function pledge(uint256 _id, uint256 _amount) external {
        Campaign memory campaign = campaigns[_id];
        require(block.timestamp >= campaign.startAt, "not started");
        require(block.timestamp <= campaign.endAt, "ended");

        campaign.pledged += _amount;
        pledgedAmount[_id][msg.sender] += _amount;
        token.transferFrom(msg.sender, address(this), _amount);

        emit Pledge(_id, msg.sender, _amount);
    }
```

**Do we see any issues?**

Appears to be ok, but let's examine `campaign.pledged += _amount;`

What happens if we have another function that checks the total amount pledged.Well, the function would return zero.

This is because, even though we do increment the pledged amount whenever `pledge()` is called, this is never stored in storage. we’re modifying a copy in memory, not the actual stored data. No error is thrown but nothing gets saved either.

The line ` Campaign memory campaign = campaigns[_id];` is where the issue stems from.

To fix this we just need to ensure this is stored in storage;

```solidity
Campaign storage campaign = campaigns[_id];
```

[See correct implementation on solidity-by-example](https://solidity-by-example.org/app/crowd-fund/)

These are just some of the ways lack of storage update can manifest. see the following report for another interesting case:
[Whitelisted accounts can be forcefully DoSed from buying `curveTokens` during the presale](https://github.com/code-423n4/2024-01-curves-findings/issues/1068)

## 🚀 Conclusion

In smart contracts, every state update matters. Forgetting to subtract, delete, or update a value can leave your protocol in a broken state, even if everything looks like it’s working. it is therefore very important to keep track of all state changes and ensure everything is being updated as required.

**Note: just because it compiles doesn't mean it works — ⚠️**
