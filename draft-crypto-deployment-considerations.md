---
title: "Deployment Considerations for Cryptographic Protocols"
abbrev: "Deployment Considerations for Cryptographic Protocols"
category: info

docname: draft-crypto-deployment-considerations-latest
submissiontype: IRTF
number:
date:
consensus: false
v: 3

author:
 -
    fullname: Christopher A. Wood
    organization: Cloudflare
    email: caw@heapingbits.net

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO intro

<!--
Prompt: What are practical considerations for deploying cryptographic protocols, especially surrounding bandwidth, rounds, memory, and computation limits? For each item, give an example about from practice.

Computation:
- There are no general constraints here. Sometimes crypto is expensive, and how much computation is too much depends on a couple of things, like the actual cost of computation in terms of battery life, financial cost, as well as the potential impact on the user experience. Minimizing is always better, but there is a point of decreasing returns, so over-emphasizing on computation typically isn't the right tradeoff.

Memory:
- Different types of memory use, including long-term memory and ephemeral short-term memory. There's also different aspects of state throughout a protocol.
- No session identifiers or global counters to track state.
- Ephemeral state is okay and modeled with protocol state machines, FROST nonce as state.
- Long term state nonce reuse state is not really feasible, eg Hash based signatures not deployable unless state space is isolated or partitioned.

Rounds:
- Fewer rounds is better. Exactly one round is best, as it does not require memory across rounds of a session.
- The concept of a session does not map to all deployment environments. HTTP is a common transport, but doesn't have the concept of a "session," and many deployments of HTTP servers do not have a concept of session unless implemented with some form of state.
- Putting state on the wire (encrypted) requires replay attack mitigations, and therefore state.

Bandwidth:
- Message sizes do have a practical limit depending on the underlying transport. They might influence transport characteristics and lead to worse performance. Sometimes the headers might not even fit in the presence of middleboxes.

Communication and coordination:
- Broadcast is bad, unicast is good. Mutually authenticated channels are good.
- Protocols that require coordination across different parties are complicated to implement and ship in practice.

Maintenance:
- Cost of implementing new cryptographic primitives is high since there's a long-term maintenance cost. Formally verified implementations help, but may not be enough.
-->

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
