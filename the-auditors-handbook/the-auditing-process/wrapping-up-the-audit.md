---
description: It's been fun
---

# Wrapping Up The Audit

### Tie Up Loose Ends

The audit is coming to an end!&#x20;

Review and resolve all `@audit` tags and make sure every last note/thought you had is resolved. Then take the time to double-check that all of your findings are in the doc and that they are adequately documented with accurate line numbers and suggested changes.



### Validate Findings

After everyone has finished their review, go through the findings doc and independently validate each finding. More complex/dubious findings can be discussed and defended as a group.&#x20;

As a part of the verification process, ensure that the recommended fix fully resolves the issue in a desirable way.&#x20;

{% hint style="info" %}
PoCs are critical for sharing and defending a finding, make sure your PoCs are legible and sufficiently commented.
{% endhint %}



### Create The Report

Now it's time to create the report!&#x20;

For each valid finding, populate a slide in the report with the description, file/line, criticality, status, and remediation. Read through the description/remediation several times to correct any typos and re-word as necessary.

For high-severity findings, link to their corresponding PoCs for more context.

Deliver the report along with the test suite repo, this way the team can refer to PoCs, see the due diligence that was done, and perhaps even adopt the test suite as they make remediations.
