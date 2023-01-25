---
description: The ideal setup and prep for an audit
---

# Audit Setup/Preparation

### Logistics

First and foremost are the logistics of an audit, it is crucial to appropriately scope the time and # of collaborators necessary for an audit. Aside from auditor expertise, the main driver of audit quality and resulting security is simply the amount of engineering hours invested.



It is always ideal to work with other auditors, no matter the size/complexity of the audit. For simple projects, two auditors is almost always enough. For larger, more complex projects, more auditors may be necessary.

A ballpark rule of thumb is: 1 auditor per \~2,000 SLOC

This is a \*rough\* estimate, for projects with specialized logic such as advanced mathematics or financial concepts this ballpark goes out the window.



Now how much time should each auditor invest?

Each auditor should strive to cover the entire codebase and corroborate their findings with their colleagues.

The often mentioned 200 SLOC/hour may apply to simple contracts or decentralized contests where missing bugs/vulnerabilities is fine, but doesn't hold for an audit of a large and complex protocol.

As the number of contracts in the system increase, the time necessary to audit each SLOC increases quadratically. For complex projects this rate may drop to 100 SLOC/hour or even 50 SLOC/hour for especially nuanced/mission critical code.



Additionally, be sure you are well rested and have a clear mind throughout the audit. Overworking and not taking care of yourself will harm mental clarity and affect the quality of your auditing.



### Research/Context

Gaining the appropriate context before beginning an audit will save crucial time at the beginning of the audit.

Learn about what the project is trying to accomplish and how it plans to accomplish that goal technically. If there is documentation for the project, pour over it before the audit begins.

Be sure to ask the team to provide any relevant documents such as specs, RFCs, or diagrams.

Learn about similar protocols that have been created and audited or exploited in the past. Read the previous audits or postmortems. E.g. if it is a DEX, research previous DEX models and familiarize yourself with the common vulnerabilities for DEX's.



### Communication

Coordinate with your fellow auditors to pick a communication medium and cadence.&#x20;

It is extremely helpful to have a group where all auditors can communicate about exploit ideas and share context on areas of the codebase.

A combination of 24/7 chat with scheduled calls to walk through leads on vulnerabilities/braindump  has proved to be effective for the team at Guardian.



### Tooling Setup

There are a few tools/resources you'll want to prepare before the audit begins that will help you throughout the process.

Firstly, you'll want to make sure you have an editor that can easily traverse the contracts, using something like IntelliJ or VSCode will allow you to click through and see the definition of each function/contract.

Next you'll want to set up a findings document that you can share amongst all the auditors collaborating. The findings document acts as a bank of all findings, separated into their respective criticality, potentially with links to their corresponding PoCs.

A Forked TestSuite repo will be helpful for collaborating on tests and PoCs.

Finally, you'll want to take time to setup any infrastructure necessary for tools like Echidna, Manticore, Foundry etc...

{% hint style="info" %}
There will be a complete separate guide on security tooling, for the time being these tools are considered out of scope for this handbook.
{% endhint %}
