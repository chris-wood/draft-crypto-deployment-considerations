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

TODO Introduction

<!--
Prompt: What are practical considerations for deploying cryptographic protocols, especially surrounding bandwidth, rounds, and computation limits?

Bandwidth:
- XXX

Rounds:
- XXX

Computation:
- XXX

State:
- No session identifiers or global counters to track state
- Ephemeral state is okay and modeled with protocol state machines, FROST nonce as state
- Long term state nonce reuse state is not really feasible, eg Hash based signatures not deployable unless state space is isolated or partitioned
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
