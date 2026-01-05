+++
draft = true
title = "Cryptid Technologies Provenance Stack – My Initial Understanding"
slug = "cryptid-provenance-stack"
date = "2026-01-04"

[taxonomies]
tags=["keri", "provenance", "cryptid"]

[extra]
comment = true
+++

# Cryptid Technologies Provenance Stack – My Initial Understanding


Dave Grantham’s work on the Cryptid Technologies Provenance Stack ([http://cryptid.tech](http://cryptid.tech)) includes the following:– Wasm Cryptographic Constructs (WACC) Virtual Machine (VM) API [Specification](https://github.com/cryptidtech/provenance-specifications/blob/main/specifications/wacc.md)

## WACC

Extends the standard WebAssembly VM with a set of data manipulation and cryptographic operations.

- A way to describe data validation scripts.
- Primary use for WACC VM scripts is to provide control over who can append events to a provenance log.

PLog – Cryptographic Provenance Log

