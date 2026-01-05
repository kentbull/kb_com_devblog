+++
title = "KERI Update: Major release and Branch Strategy Change"
slug = "keri-update-major-release"
date = "2024-04-14"

[taxonomies]
tags=["keri", "acdc", "keripy", "keria", "signifypy", "release"]

[extra]
comment = true
+++

# KERI Update: Major release and Branch Strategy Change

The WebOfTrust community recently released a major update to the Key Event Receipt Infrastructure (KERI) and Authentic Chained Data Containers (ACDC) implementation as a coordinated release across the suite of WebOfTrust repositories.

## Coordinated Release

This resulted in the following release versions:

- **Repository**: KERIpy (used for witnesses, command line-managed decentralized identities, and as the core library for the Python KERI space)
  **Version**: **1.1.12** ([PyPi](https://pypi.org/project/keri/)) – Apr 9, 2024

- **Repository**: KERIA (agent server used for individual decentralized identity agents – your data sits here)
  **Version**: **0.2.0.dev0** Pre-Release ([PyPi](https://pypi.org/project/keria/)) – Apr 11, 2024

- **Repository**: SignifyPy (edge-signing client implementation – your keys sit here)
  **Version**: **0.1.0** ([PyPi](https://pypi.org/project/signifypy/)) – Feb 13, 2024

## Branch Strategy Change

- "**development**" branch merged to "**main**" as the old GitFlow style branching strategy was dropped in favor of trunk-based development (single main branch strategy). See the [keripy#726 GitHub discussion](https://github.com/WebOfTrust/keripy/discussions/726) for the rationale.
  - This occurred for the following repositories:
    - Python
      - KERIpy
      - KERIA
      - SignifyPy
    - Typescript
      - SignifyTS

## Recent Blog Posts

See Nuttawut Kongsuwan's explanation of how to use KERI in his ["The Hitchhiker's Guide to KERI. Part 3: How do you use KERI?"](https://medium.com/finema/the-hitchhikers-guide-to-keri-part-3-how-do-you-use-keri-e2c71a296f00)
