# Decision Tree

```text
Incident reported
|
+-- Is there direct evidence?
|   |
|   +-- No --> Request minimal high-value evidence
|   |
|   +-- Yes
|       |
|       +-- Classify affected layer
|       +-- Generate multiple hypotheses
|       +-- Rank by evidence
|       +-- Choose safest next action
|       +-- Validate
|
+-- Is the proposed action risky?
    |
    +-- Yes --> Require rollback and explicit approval
    +-- No  --> Execute and measure
```
