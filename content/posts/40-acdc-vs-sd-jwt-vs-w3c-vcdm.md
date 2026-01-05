+++
draft = true
title = "ACDC vs. SD-JWT vs. W3C VCDM 1.0 and 2.0"
slug = "acdc-vs-sd-jwt-vs-w3c-vcdm"
date = "2026-01-04"

[taxonomies]
tags=["acdc"]

[extra]
comment = true
+++

# ACDC vs. SD-JWT vs. W3C VCDM 1.0 and 2.0

ACDCs unique features:

- Credential Chaining
- Multisig (KERI)
- Secure
- Are anchored to

SD-JWT:

SD in SD-JWT is selective disclosure.

- Phone home?
- Does not chain
- Graduated disclosure is not a part of SD-JWT
- Rules sections not present

W3C VCDM

- 1.0 and 2.0

From Kevin:Features ACDC Has (SD-JWT Doesn’t Have)

– Cryptographic chaining between credentials (DAG structure)

– Self-addressing identifiers (SAIDs)

– Multiple serialization formats (JSON, CBOR, MGPK, CESR)

– Graduated disclosure

– Contractually protected disclosure

– Compact field labels for IoT/constrained environments

– Built-in schema validation system (Schema as SAID)

Features SD-JWT Has (ACDC Doesn’t Have)

– Direct JWT compatibility and ecosystem integration (potentially addressed by VRD extension)

– Standard OAuth/OIDC integration (OIDC4VC) (IPEX is simpler but not standard in EU)

– Simpler implementation (extends existing JWT libraries, sigh)

– Direct support for existing JWT verification tools (I guessed at this)

Shared Features?

– Selective disclosure of individual data elements

– Digest concealment of undisclosed data

– Privacy-preserving credential presentation

– Salt-based randomization for unlinkability

I spent less than 30 minutes reading the SD spec.

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
