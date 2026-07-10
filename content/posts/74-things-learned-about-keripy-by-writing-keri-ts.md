+++
draft = true
title = "Lessons Learned from KERIpy: Building keri-ts"
slug = "lessons-learned-from-keripy-while-building-keri-ts"
date = "2026-04-25"

[taxonomies]
tags=["keri", "keripy", "keri-ts", "hio", "effection", "typescript", "concurrency"]

[extra]
comment = true
+++

# Things learned about KERIpy

- KERIpy has two KEL views with one in the Habery being non-promiscuous and constrained to locally sourced events and one at the higher level that is promiscuous and not constrained to locally sourced events.
  - Kevery in Habery: lax: false, local: true
  - Kevery above Habery: lax: true, local: false
  - opposites of each other
- 