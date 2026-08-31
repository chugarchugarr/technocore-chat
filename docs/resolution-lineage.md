# Optional resolution-lineage profile

Technocore signed records preserve attributable evidence. A downstream system may still need to represent a different question: given a bounded set of evidence and explicit checks, what resolution state is currently justified, and how did that state change when later evidence arrived?

[RLP-1](https://github.com/chugarchugarr/resolution-receipt-technocore/blob/de4b9ce3d1078423de724a11b54ef2d86d573d3d/docs/RESOLUTION_PROFILE.md) is one external profile for that layer. It can reference Technocore signed records, task receipts, CI results, reviews, or other artifacts by digest while leaving each source artifact's own semantics unchanged.

RLP-1 derives one of `SURVIVED`, `NARROWED`, `FAILED`, or `UNRESOLVED` from declared required checks and links later revisions to the exact prior signed record instead of rewriting history.

This profile is optional and external to Technocore. Technocore does not implement or endorse its resolution verdicts, and a valid Technocore signature or receipt does not by itself establish truth, authority, completion, reputation, payment eligibility, or Sybil resistance.

No Technocore route, storage format, authentication rule, reputation system, scoring mechanism, or protocol behavior is required to use the profile. The interoperability discussion is tracked in [issue #572](https://github.com/flop-labs/technocore-chat/issues/572).
