+++
draft = true
title = "KERI Internals Part 2: Core Data Structures – KEL Events, TEL Events, and Seals"
slug = "keri-internals-data-structures"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERI Internals Part 2: Core Data Structures – KEL Events, TEL Events, and Seals

How do components in a KERI ecosystem communicate to one another? Through exchange messages.

- Make a diagram of the data structures and how they relate from interaction event to management TEL to ACDC TEL and to ACDC, then between ACDCs. Show the whole graph.
- Show the precise events that occur for:

Management TEL creation
- ACDC TEL creation for issuance
- ACDC TEL revocation
- Multisig coordination: EXNs
- Challenge Response

- Charles Lanahan: Are xip/exn for sessions, qry/rpy for async non-private data, prod/bare for async private (sealed) data?

The core events in the KERIpy include:

**Event Messages**

1. icp – Inception Event
2. rot – Basic Rotation Event (also delegating rotation)
3. ort – Partial Rotation Event
4. isn – Interaction Event
5. dip – Delegated Inception Event
6. drt – Basic Delegated Rotation Event
7. dor – Delegated partial Rotation Event

**Receipts**

1. rct – Receipt
2. vrc – Transferable Prefix Signer Receipt

**Other Messages**

1. qry – Query Message
2. rpy – Reply Message
3. exp – Expose Message
4. exn – Exchange Message

The “why” behind the “rp” field: (from the July 21, 2024 dev call)

In an exn message there is a “rp” field for the recipient even though in the “a” attributes of an exn message there is a recipient field, the “i” field, because there are use cases such as in ESSR where the “a” payload is externalized and is encrypted thus making the “i” field not usable.

The “i” field is named “i” because it came from ACDCs, the issuee, and the old label is preserved for backwards compatibility.

- The “why” behind the “rp” field: (from the July 21, 2024 dev call)

In an exn message there is a “rp” field for the recipient even though in the “a” attributes of an exn message there is a recipient field, the “i” field, because there are use cases such as in ESSR where the “a” payload is externalized and is encrypted thus making the “i” field not usable.

The “i” field is named “i” because it came from ACDCs, the issuee, and the old label is preserved for backwards compatibility.

**Notices Embedded in Reply Messages**

1. ksn – Key State Notice
2. tsn – Transaction State Notice

**Transaction Event Log Messages**

1. vcp – Registry Inception Event
2. vrt – Registry Rotation Event
3. iss – Backerless Credential Issuance
4. rev – Backerless Credential Revocation
5. bis – Backer Credential Issuance
6. brv – Backer Credential Revocation

**Seals**

1. d – Digest Seal
2. rd – Merkle Tree Root Digest Seal
3. d + bi – Registrar Seal
4. d + i + s – Event Seal

KERI Event Configuration Traits

- EO – Establishment Only
- DND –
- NRB
- RB

ACDC m-ary Edge Operators

- AND
- OR
- NAND
- NOR
- AVG
- WAVG

ACDC unary Edge Operators

- I2I
- NI2I
- DI2I
- NOT

IPEX Exchange Protocol

- apply
- spurn
- offer
- agree
- grant
- admit

## References

[1] KERI Message and Seal Formats, Samuel Smith, [https://hackmd.io/XfdKjT3ZQDi1M6Iv3iYhbg](https://hackmd.io/XfdKjT3ZQDi1M6Iv3iYhbg)

[2] Transaction Event Logs / PTEL md [https://github.com/WebOfTrust/keripy/blob/main/ref/tel.md](https://github.com/WebOfTrust/keripy/blob/main/ref/tel.md)

[3] PTEL spec [https://trustoverip.github.io/tswg-ptel-specification/draft-pfeairheller-ptel.html](https://trustoverip.github.io/tswg-ptel-specification/draft-pfeairheller-ptel.html)

