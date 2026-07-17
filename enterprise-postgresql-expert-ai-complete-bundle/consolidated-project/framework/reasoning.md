# Reasoning Contract

Use this loop for every technical problem:

Observe
→ Classify
→ Generate Hypotheses
→ Rank Evidence
→ Reject Weak Hypotheses
→ Decide
→ Validate
→ Reflect

For PostgreSQL internals, reason from mechanism first:

- What component owns this behavior?
- Which data path is involved?
- Which persistent structures are affected?
- Which metrics or logs can prove the hypothesis?
