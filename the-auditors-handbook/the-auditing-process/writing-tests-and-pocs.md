---
description: Time to test!
---

# Writing Tests & PoCs

### Tests

If the test coverage is poor, fill in the gaps. By writing tests, you get a more intimate understanding of the contracts + there’s a good chance you find a bug (untested code is hearsay). There are some bugs that are much more obvious to a runtime execution than to human manual analysis.

Test suites should aspire to reach 100% code coverage, the behavior of untested code paths is dubious.

While you're writing tests, you may encounter some odd behaviors that you didn’t realize before — `@audit` tag them and explore these further after you finish your current thought/test.

### PoCs

Now take the time to PoC any findings/attack vectors that you haven't already. It’s important to comment throughout each PoC sufficiently, both for your own understanding and others.

While writing a PoC, you might discover that the system does not function as you thought it did, and your attack is not viable. In this case, go back to the drawing board and examine how your attack could be tweaked (the list of knobs is helpful here) so that the attack is valid.

{% hint style="info" %}
Now is a good time to use security tools to verify invariants you identified during the previous stage, etc... ("Security Tools" under construction)
{% endhint %}
