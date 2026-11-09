+++
title = "keri-ts"
description = "TypeScript implementation of KERI, CESR, and ACDC, with the Tufa command line"
weight = 8
date = 2025-02-23

[extra]
github = "https://github.com/kentbull/keri-ts"
tags = ["keri", "cesr", "acdc", "typescript"]
+++

A TypeScript monorepo for KERI, CESR, and ACDC. It publishes the `keri-ts` protocol and runtime library, the `cesr-ts` parsing and annotation library, and `@keri-ts/tufa`, the Trust Utilities for Agents command line. Install the CLI with `npm install -g @keri-ts/tufa`, then use `tufa annotate` to turn a raw CESR stream into something you can read.
