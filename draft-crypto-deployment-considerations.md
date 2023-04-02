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

Many real world problems require implementing and deploying cryptography as
part of the solution. In general, there is no single standard or set of requirements
by which applications determine what type of cryptographic solution is best for
their problem. Different applications and deployments can lead to varying tradeoffs
in computation, memory, network, and bandwidth properties of a solution. Moreover,
practical aspects of modern software engineering, especially around long-term
maintenance costs, may influence what type of cryptographic solutions are deployed
in practice. This document attempts to cover different factors that influence
what type of cryptography is deployed in practice with the goal of helping
cryptographic researchers navigate the tradeoffs and assumptions made in new
and emerging work.

--- middle

# Introduction

Software engineering is like any other form of engineering; every problem that
needs a solution requires careful cost-benefit analysis to determine what type
of solution is best. Naturally, there is rarely one single best answer. Engineers
make tradeoffs when navigating the solution space as a way of balancing a variety
of real world constraints and requirements.

Constraints and requirements vary widely in practice. For example, constraints
can apply to the specific properties of a solution, such as the amount of computation
or network round trips required by a particular cryptographic protocol. Constraints
can also apply to the threat model and trust assumptions to a soluton, such as
whether or not different parties in the system are considered trusted or not.
Finally, constraints can also apply to pragmatic engineering considerations beyond
the functional properties of a cryptographic solution, such as the long-term
maintenance costs of implementing a particular algorithm or solution.

There is no single standard or set of requirements by which applications determine
what type of cryptographic solution is best for their problem. However, awareness
of these real world requirements or constraints can help in navigating the solution
space by designing cryptographic algorithms or protocols. In particular, understanding
the considerations that apply to real world deployments of cryptographic solutions
can help narrow the design space for a particular problem. It can also help ease the
transition from novel cryptographic research into practice.

The intent of this document is twofold. The first objective of this document is to
discuss constraints and requirements that are factored into the implementation and
deployment of real world cryptographic solutions. These considerations may not be
exhaustive, and contributions which extend this list of considerations are certainly
welcome. The second objective of this document is to survey concrete examples of
deployed cryptography to illustrate how certain tradeoffs with respect to these
constraints and requirements were made.

The target audience of this document is cryptographic and security researchers who
are working on solutions to problems that exist in the real world.

<!--
Prompt: What are practical considerations for deploying cryptographic protocols, especially surrounding bandwidth, rounds, memory, and computation limits? For each item, give an example about from practice.

Communication and coordination:
- Broadcast is bad, unicast is good. Mutually authenticated channels are good.
- Protocols that require coordination across different parties are complicated to implement and ship in practice.

Maintenance:
- Cost of implementing new cryptographic primitives is high since there's a long-term maintenance cost. Formally verified implementations help, but may not be enough.
-->

<!-- 
Real World Examples: iCloud Private Relay, Private Access Tokens, GeoKDLv2, Prio (ENPA), STAR, QUIC, WireGuard, ZCash(?), WhatsApp E2E encrypted backup (no threshold OPRF)
-->

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Engineering Constraints

Engineering is a balancing act of tradeoffs with the end goal of providing high value
solutions. For problems that require (in part) cryptographic solutions, balancing the
different real world constraints well is particularly important. Improper or incorrect
solutions can be costly in the best case or have security vulnerabilities in the worst
case.

This section discusses constraints that influence how these tradeoffs are made in
practice. XXX(caw): finishme

## Functional Constraints

Functional constraints determine what functional properties of a solution are feasible
for a particular deployment. The functional properties of a cryptographic solution
generally include the amount of computation, memory, network round trips, and network
bandwidth needed for a particular solution. These properties all contribute to the overall
quality of a solution. For example, in applications which have real-time requirements,
such as web browsing, excessive computation means that a solution takes longer to complete,
which may negatively influence the user experience.

In general, functional properties affect the cost of a solution, where cost can be
measured in terms of financial expense, battery life or power consumption, or cost to
the end user experience. Solutions seek to minimize their cost while maximizing their
quality. 

This section describes how these properties are constrained in practice to minimize cost
and provides examples of how these constraints influenced other relevant quality metrics.

### Computation

Minimizing computation is generally always better, but only up to a point. For example,
consider the problem of authenticated key exchange with TLS. The actual key exchange step
generally involves some public key cryptography operations. When used in the context of a
user-facing application, such as web browser, a desired cryptographic solutions to the key
exchange step should be less than the amount of time that a human can perceive any latency,
as otherwise the cost of this step can negatively affect the user experience. However,
minimizing the cost of this step is only useful up to the point where the cost of network
round trips exceeds that of the key exchange computation. In other words, at a certain point,
the cost of computation is no longer the bottleneck.

As another example, sometimes cryptographic solutions can minimize computation as a way
of decreasing financial cost to deploy the solution, but this too has a limit. For example,
consider the case of privacy-preserving measurement using {{?STAR=I-D.dss-star}}. STAR
is a very lightweight and efficient protocol for measuring heavy hitters. However, this
efficiency comes from a relaxation of the threat model. STAR is most cost efficient when
it assumes honest clients and honest servers, as otherwise there is no need to do additional
checks to mitigate abuse by malicious parties. STAR was motivated by the need to decrease the
cost of measuring heavy hitters through less computation, yet at the cost of a weakened
threat model compared to alternative solutions.

As another example, consider the case of batched zero knowledge proof generation and verification
for cryptographic constructions based on VOPRFs. Solutions may wish to minimize the cost of proof
generation and verification across many independent requests as a way of decreasing overall
computation. However, in practice, an implementation of this solution is more complex than
one in which there is no coordination across independent requests, in particular because the
VOPRF signer needs to implement batching and the logic necessary to handle batch processing
in a timely manner. Thus, this decrease in computation comes at the cost of implementation
complexity.

### Memory

Minimizing memory consumption -- or state -- is an advantageous goal. However, it's important
to distinguish what types of memory or state a particular proposes when exploring this
constraint. There are effectively two types of memory or state that are relevant for
cryptographic solutions: 

1. Long-term state, such as private keying material that exists for repeated runs of a
cryptographic protocol.

1. Ephemeral state, such as one-time-use key shares and random nonces used for a key
exchange protocol, that only exists for the duration of a given cryptographic protocol
and is effectively captured or constrained within the state machine of a protocol.

Minimizing all three types of state or memory is generally infeasible; every solution
inevitably requires some form of state. Moreover, there is generally no best answer
for which type of state to minimize. Considerations that apply to each are below.

1. Long-term state. Minimizing this type of state simplifies management. For example,
   in the case of long-term private keys, minimizing the number of keys can simplify
   mechanics that need to be in place to deal with issues such as revocation and rotation.
   Fewer keys may need to change as a result. However, minimizing such long-term state
   means that each key gets more use across all possible users. In the extreme case where
   there is just a single key, the key becomes a shared resource with access contention.
   Requests to use the key can therefore lead to decreased performance, especially if
   access is protected by a mutex or similar. In comparison, where there is one key per
   possible user, this contention may not exist, but managing these keys may require
   more complicated key management machinery.

   Minimizing long-term state and the contention that results can also manifest in security
   vulnerabilities if contention is not dealt with appropriately. For example, some hash-based
   signature schemes require that no nonce ever be reused, as otherwise the private signing key
   is immediately revealed. Minimizing the number of private signing keys increases the need
   for contention, since if the signer does not properly track signed nonces then key compromise
   is possible. Some cryptographic protocols assume techniques to manage such state exist, yet
   for practical reasons are not very realistic. Some threshold signature protocols, for example,
   may assume that signature nonces are tracked globally across all signing participants, but
   implementing such a strongly consistent database for is challenging in practice. The CAP
   (consistency, availability, partition-tolerance) theorem [TODO] strongly suggests that such
   assumptions are impractical for protocols that require availability, since typically
   consistency and partition-tolerance are necessary for security.

   In general, long-term state is feasible to maintain provided there is no consequence
   for contention, be it a performance consequence due to mutual access or a security
   consequence to failure to properly manage this state.

1. Ephemeral state. Minimizing this type of state simplifies the protocol state machine.
   As such, minimizing this is useful insofar as it helps minimize the overall cost
   of the protocol. A "stateless" protocol, i.e., one in which the state machine does not
   encode state across rounds of interaction, is generally the best outcome, as it means
   that there is no need to track state across rounds of interaction. A "stateful" protocol
   is one in which the state machine does require state that extends beyond rounds of
   interaction.
   
   There are significant practical differences between stateful and stateless protocols.
   In particular, a stateless protocols maps very cleanly to stateless transport protocols
   such as HTTP, making them much easier to deploy in modern application environments than
   stateful counterparts. While stateful protocols can be implemented using HTTP as transport,
   this requires either storing and managing protocol state in a local database. (An alternate
   strategy might be to store state "on the wire," where this state is encrypted for only one
   party and then transferred between parties, but this requires replay protection and thefore
   yet again some form of local database.)
   
   Not all deployment environments offer some form of local database. Moreover, even for those
   that do, the consistency guarantee of the database may not be that which is necessary for
   the protocol.

### Round Trips

The benefit of reducing protocol rounds depends on factors. As described in {{memory}}, minimzing the
number of rounds to exactly one has the benefit of producing a stateless protocol, thereby easing
deployment. Assuming all other functional characteristics (computation cost, memory, etc) stay the same,
reducing the number of rounds decreases the performance profile of the protocol. 
However, often reducing the number of rounds negatively affects other aspects of the protocol.
For example, reducing rounds may require more bandwidth per round, more complicated implementation
or cryptography in order to provide the same functionality, or it may require a weaker threat model.

As an example, some protocols are specified with a notion of preprocessing, wherein one or
more parties do some amount of work a priori, either locally or in collaboration with other
parties, in order to simplify the main online protocol. As an example, the {{?FROST=I-D.irtf-cfrg-frost}}
threshold signature protocol consists of two phases, one of which can be done offline by
signing parties in a preprocessing phase. Splitting protocols into an offline and online
phase can help reduce the cost of the online phase, but necessarily requires coordination
between the offline and online phases. As described in {{memory}}, technologies for
coordination may or may not be available for a particular deployment environment.

As another example, some applications of zero knowledge proofs involve an offline phase that's
done to minimize the cost of proof generation in the online phase. This split is done for
practical reasons: proof generation without precomputation can be expensive. However, implementations
that use such precomputation are almost always more complicated in practice. Precomputation
is done by some process that's running in the background and separated from processes that
wish to use the output of this precomputation in the online phase. This means there must now
exist some form of IPC between these processes, and, additionally, requires that the process
implementing the online phase implement some form of input validation.

As a final note, there is little difference between two or more rounds in a stateful protocol
from an implementation perspective. Any protocol which requires two rounds must necessarily
have some mechanism for dealing with state across the rounds, and this mechanism can also be
used for storing state across any subsequent rounds. Thus, practically speaking, unless there
are performance reasons to do so, optimizing a protocol with more complicated cryptography to
reduce the number of rounds from more than two to exactly two is often not a desirable tradeoff.

### Bandwidth



<!-- Bandwidth:
- Message sizes do have a practical limit depending on the underlying transport. They might influence transport characteristics and lead to worse performance. Sometimes the headers might not even fit in the presence of middleboxes. -->

## Implementation and Maintenance Constraints

XXX: new code, new attack surface, formal verification, long-term maintenance, new API surface, etc
XXX: time pressure to ship, minimize complexity to get something out the door

## Runtime Constraints

XXX: no synchronized clocks, 

## Ecosystem Constraints

XXX: reuse and adoption is better than new stuff

## Usability Constraints

XXX: user stuff is hard...

# General Recommendations

XXX: recommentations for researchers (always choose simplicity of implementation, maximize reuse where possible, collaborate openly)

# Security Considerations

This document discusses considerations relevant to the implementation and deployment
of cryptography in practice. The document is effectively a collection of security
considerations.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document was motivated by discussions that took place during the Real World
Crypto 2023 conference.
