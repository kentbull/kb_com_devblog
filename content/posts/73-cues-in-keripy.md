+++
draft = true
title = "Cues in KERIpy - Flow Based Programming"
slug = "cues-in-keripy"
date = "2026-04-25"

[taxonomies]
tags=["keri", "keripy", "hio", "flow-based-programming", "concurrency", "queues", "cues"]

[extra]
comment = true
+++

# Cues

What are cues in KERIpy?
They are requests for critical side-effects created by a caller to schedule asynchronously later completion of a multi-step process.
Cues are side effects to be complete in the future. They are a data contract between loosely coupled components that allow Kevery, Tevery, Revery, and Exchanger to 
not have to know about transport, mailbox, query scheduling, or UI/logging behavior.

