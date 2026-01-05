+++
draft = true
title = "Building a secure contacts module for a KERI  wallet"
slug = "keri-wallet-contacts-module"
date = "2026-01-04"

[taxonomies]
tags=["keri", "wallet", "security"]

[extra]
comment = true
+++

# Building a secure contacts module for a KERI  wallet


Outline:

1. Pull the contacts implementation out of KERIpy.
2. Show how to build an arbitrary data module for a KERI wallet / agent. Sign the data coming in, only signed data comes out.
3. Show how to verify data coming out of a KERI wallet
4. Contacts should be a separate library, not something core to KERIpy. KERIpy is just keys, AIDs, KELs, CESR, and anchors.

