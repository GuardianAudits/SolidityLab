---
description: The audit begins...
---

# Beginning The Audit

Examine The Repo

The best place to start is almost always `README.md`, if there is none (or its a default generated `README.md`) then the second thing to do is scan the `.sol` files for block comments that explain technical design/gotchas.

If you read something that you think might yield an attack vector in the documentation, add a comment next to it and come back to re-evaluate it later. Adding specific tags such as `@audit` are helpful for grepping the codebase to be able to find all of your comments.

Next, examine the test coverage for any gaps, this will give you a good idea of the quality of code you're dealing with and may point you towards some good areas to focus on once you get into the meat of the audit.

{% hint style="info" %}
[solidity-coverage](https://www.npmjs.com/package/solidity-coverage) is a helpful package for tracking the code coverage of a hardhat test suite.&#x20;
{% endhint %}



### Build A Mental Model

Your mental model of the smart contract system will lay the foundation for the audit, it is critically important that you have a solid high-level understanding of the contracts before examining the nitty gritty details. Context is key when examining each individual line.

Leverage you're prior research about the protocol and enumerate the JTBD (Jobs-To-Be-Done) of the system. Does it enable swapping from one token to another? Can a user swap with multiple tokens in the swapRoute? Are users able to provide liquidity?

After you've listed the high level functionality of what the contracts aim to do, exlplore the code-paths for each of these. How does someone swap? How does someone provide liquidity?

You have sufficiently walked a code-path when you are able to describe the functions/contracts a user's tx interacts with without reading the code. If this is a DEX: how does a swap execute?

{% hint style="info" %}
Do not memorize the lines in each function, just have an idea of what each function accomplishes and the overall call-path.
{% endhint %}

As you do this initial skim of the contracts, you'll likely notice things that you don't understand at first or that seem off, add an `@audit` tag with your thoughts and come back to it later when you have more context.

<figure><img src="../../.gitbook/assets/Screenshot 2023-01-24 at 6.06.34 PM.png" alt=""><figcaption><p><code>@audit</code> tags in the code </p></figcaption></figure>

{% hint style="info" %}
It is extremely important to make note of everything that seems off, when exploring a large codebase it is easy to become overwhelmed and forget each code-smell you had.
{% endhint %}

As you read and have questions, reach out to the other auditors, they may have insights or be wondering the same thing. Combining your mental model with the other auditors drastically speeds up the process of gaining context and saves precious time during the audit.

Sometimes it may be helpful to make a call graph with tools like [Surya](https://github.com/ConsenSys/surya). Call graphs can help visualize the system and give context as you're walking through the code path, they can also serve as an aid when communicating with other auditors.&#x20;

{% hint style="info" %}
When using a call graph, it may be especially useful to mark the contracts that hold vital storage and the contracts that hold funds.
{% endhint %}

