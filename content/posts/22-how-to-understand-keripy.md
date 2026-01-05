+++
draft = true
title = "KERI Series: How to understand KERIpy – the reference implementation"
slug = "how-to-understand-keripy"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERI Series: How to understand KERIpy – the reference implementation

Flow based programming.

Coupling between parser and core business logic components.

Escrows

LMDB – DupSort, Insertion ordered keys, keyspace vs value space

Events

Perceptions people have of KERIpy:

- CESR spec is huge. Why is it needed?
- AID is kind of like a DID, yet not quite the same.
- Understand witness, yet lost on the need for a watcher and an observer

Answers:

- witnesses do one thing, just receipt events that you send them
- Watchers are a way of publishing a key event log, yet not your own. You watch someone else’s KELs (local watcher)
- Observer

