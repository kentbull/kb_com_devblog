+++
draft = true
title = "KERI: What is BADA-RUN and how does it work?"
slug = "keri-bada-run"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERI: What is BADA-RUN and how does it work?

Describe how the Best Available Data Acceptance policy works and why it matters.

Show the ToIP internet hourglass architecture model with the four following thin layers, in order:

- OOBI Discovery
- Reply Messages
- BADA-RUN
- KERI

See the [BADA (Best-Available-Data-Acceptance) Policy section](https://trustoverip.github.io/tswg-keri-specification/#bada-best-available-data-acceptance-policy) in the ToIP KERI [specification](https://trustoverip.github.io/tswg-keri-specification) near the end of the document.

Main points:

1. Peers create and maintain their own records and data.

This means updates are signed by each peer.
2. Current database state is an aggregate of submissions of all peers.

Similar to how the Git source control system works.
3. Local-first, offline, and peer-to-peer communication friendly.

- This means updates are signed by each peer.

- Similar to how the Git source control system works.

{% image(path="/images/posts/28-bada-run-diagram.png", class="diagram",
    op="fit_width",
    alt="BADA-RUN Architecture Diagram") 
    %}
BADA-RUN Architecture Diagram
{% end %}

- Actions

Create
- Read
- Update
- Delete

- Data hosting role (originates identity, source of truth)
- Data source role (often the same, API or DB host, in client/server, yet different in Git)
- Data consumer role (often the same as source role, API or DB host, in client/server, yet different in Git)

[visualization of peer-oriented architecture]

- Actions

Read (git pull)
- Update (git commit and git push)
- Nullify (git commit and push a deletion)

- Data hosting peer (replicates data among peers)
- Data source peer (user-controlled node that sources data)
- Data consumer peer (user-controlled node that receives data from sources)

### Unanchored data visualization with signature

[picture]

[Visualization of Signed (non-anchored) updates]

[Visualization of KEL Anchored Updates]

## Use Cases in KERI

### OKEA – Discovery of KERI components via OOBIs

What is an OOBI?

An OOBI is an out of band identifier. Out of band is a term that requires context in order to understand. It typically refers to messages exchanged through a primary communication channel. Everything shared in this primary communication channel is considered “in band” and everything outside of that communication channel is considered “out of band.”

What is an OKEA?

An OKEA is an OOBI KERI Endpoint Authorization that uses BADA-RUN for endpoint disclosure. What is an endpoint? It is a URL that indicates a route, typically accessible as a REST route over HTTP, though may be gRPC or other transport mechanism.

[show example of endpoint URL]

[show example of OKEA]

Punch line:

> OOBIs leverage the existing Internet discovery mechanisms but without needing to trust the Internet security model (or the lack of one). End-verifiability in KERI provides safety to any OOBI discovery. The Internet’s discovery mechanism, DNS/CA, is out-of-band with respect to KERI security guarantees. Thus, OOBIs may safely use DNS/CA, web search engines, social media, email, and messaging as discovery mechanisms. The worst case is the OOBI fails to result in a successful discovery and some other OOBI must be used. ToIP KERI Spec Section 9.7.6

OOBIs leverage the existing Internet discovery mechanisms but without needing to trust the Internet security model (or the lack of one).

End-verifiability in KERI provides safety to any OOBI discovery. The Internet’s discovery mechanism, DNS/CA, is out-of-band with respect to KERI security guarantees.

Thus, OOBIs may safely use DNS/CA, web search engines, social media, email, and messaging as discovery mechanisms.

The worst case is the OOBI fails to result in a successful discovery and some other OOBI must be used.

Let’s break this quote down.

A Principal Controller in the Signify model (signing at the edge) uses various KERI components including an Agent Controller. To interact with the Principal one must go through the Agent. How does one know the Agent has been authorized by the Principal? Through  BADA-RUN logic.

Purpose: securely manage endpoint data created by a Principal Controller.

[show sequence chart of successful discovery with signed **reply** message]

[show exchange of both /end/role and /loc/scheme messages in response to OOBI share or authorization]

