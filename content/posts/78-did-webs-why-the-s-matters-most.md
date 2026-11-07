+++
title = "Talk: did:webs, Why the 'S' Matters Most"
slug = "did-webs-why-the-s-matters-most"
date = "2026-11-06"

[taxonomies]
tags=["keri", "acdc", "did-webs", "did", "talks", "conference"]

[extra]
comment = true
+++

The recording of my talk from the [KERI Mini Conference, November 2026](https://keri.foundation/confs/KERI/2026/nov/) is now up on keri.foundation:

**[did:webs, why the 's' matters most](https://keri.foundation/confs/KERI/2026/nov/#kent-bull-did-webs-why-the-s-matters-most)** (59 minutes, with video, audio, and a [full transcript](https://keri.foundation/confs/KERI/2026/nov/transcripts/html/talks/kent-bull-did-webs-why-the-s-matters-most.html)).

<!-- more -->

## The short version

`did:web` is simple: put a `did.json` on a web server and your URL becomes your DID. That simplicity is why so many people reached for it. The problem is what happens when your keys change. `did:web` just swaps the published keys, so the link between your old keys and your new keys is implied by the URL, never proven.

The "s" in `did:webs` fixes that. Alongside the `did.json`, a `did:webs` host serves a `keri.cesr` stream holding the identifier's key event log and a few ACDCs. With those, a verifier can check for itself that every key rotation belongs to the same KERI identifier and that the identifier controls the domain it claims. You don't have to take the web server's word for it. Everything you know about `did:web` still applies; `did:webs` adds just enough on top to make it verifiable with KERI and ACDC.

## What the talk covers

- Resolving a `did:webs` DID, and how the DID document is derived from KERI key state and a designated aliases ACDC
- Keeping the same identifier when keys rotate or when hosting moves to a new domain
- Multisig issuers and holders, expressed through `ConditionalProof2022`
- Where the spec stands: version 0.10.3, open review points from the KERI stack working group, and the KERI 2 / ACDC 2 work still ahead (spec version 1.1, formerly known as KERI 2)
- The path to becoming a DIF recommended DID method, and the second implementation in TypeScript
- Where to find the reference `did:webs` resolver and the `did-webs-ts` library

## Where things stand

`did:webs` works end to end today with KERI and ACDC version 1, including W3C presentations. Single-signature and flat multisig identifiers are straightforward. Nested weighted thresholds still need a separate credential proof spec and library before verifiers can interpret them easily. KERI 2 support is the largest remaining piece of work.

Any KERI identifier, transferable or not, can become a `did:webs` DID. The only extra decision is which domain to use, which you record in a self-issued designated aliases ACDC.

If you want to dig in, start at [didwebs.info](https://didwebs.info), watch the [talk](https://keri.foundation/confs/KERI/2026/nov/#kent-bull-did-webs-why-the-s-matters-most), and reach out with questions. Thanks to GLEIF for supporting the `did:webs` work.
