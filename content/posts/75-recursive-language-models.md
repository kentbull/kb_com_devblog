+++
draft = true
title = "Recursive Language Models"
slug = "Recursive Language Models"
date = "2026-04-25"

[taxonomies]
tags=["keri", "keripy", "hio", "effection", "typescript", "concurrency"]

[extra]
comment = true
+++

Main points:
- Ability to programmatically execute sub-LLM calls (narrower, context-specific LLM calls) is new.
  Existing behavior only makes sub-LLM calls after they have been fully verbalized (instantiated in output tokens).
- Self Reflective Learning Models for selection of best prompt for RLM layer given uncertainty
  analysis.

References:
- [Recursive Language Models - Jan 28 2026 v2](https://arxiv.org/pdf/2512.24601)