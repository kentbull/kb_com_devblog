+++
title = "Talk: KERIpy Async Architecture"
slug = "keripy-async-architecture"
date = "2026-11-09"

[taxonomies]
tags=["keri", "keripy", "hio", "effection", "concurrency", "talks", "conference"]

[extra]
comment = true
+++

The recording of my talk on the runtime underneath KERIpy, from the [KERI Conference, April 2026](https://keri.foundation/confs/KERI/2026/apr/), is up on keri.foundation:

**[KERIpy Async Architecture](https://keri.foundation/confs/KERI/2026/apr/#keripy-async-architecture-kent-bull)** (44 minutes, with video, [slides](https://keri.foundation/confs/KERI/2026/apr/slides/Kent%20Bull%20%EF%BD%9C%20KERIpy%20Async%20Architecture%20%EF%BD%9C%20KERI%20Conference%202026.pdf), and a [full transcript](https://keri.foundation/confs/KERI/2026/apr/transcripts/html/talks/keripy-async-architecture-kent-bull.html)).

<!-- more -->

## The short version

KERIpy runs on [HIO](https://github.com/ioflo/hio), Sam Smith's structured concurrency runtime. If you plan to read or extend KERIpy, you need a mental model of how it executes. This talk tries to give you one, good enough to work in the code yourself or to direct an AI assistant through it.

Structured concurrency makes program flow follow lexical scope. Child tasks finish or get cancelled before their parent returns, and failures and cancellation move up the task tree in a predictable way. Promises, callbacks, and Go's `go` statement all give that up, in the same way `goto` once did. Nathaniel J. Smith makes the full case in [Go statement considered harmful](https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/), which both HIO and Effection cite as inspiration. Read it.

## HIO in three concepts

- **Doist** is the root scheduler. It keeps its own clock and holds the only `sleep` in the system. A KLI command runs its tasks 32 times a second.
- **Doer** is the basic task, a non-blocking coroutine with `enter`, `recur`, and `exit` lifecycle functions. Returning `True` from `recur` marks it done.
- **DoDoer** is both a task and a group of tasks, and you can add or remove children while it runs.

## What the talk covers

- Tracing `kli incept` from its entry point through `runController`, `Doist.do`, and down to the generator `send` that drives each task
- Why `doing.doify` shows up all over KERIpy: `DoDoer.recur` doesn't support `yield from`, and `doify` lets sequential steps stay in one function. I think DoDoer should support it. Sam prefers separate Doers, and there is a real trade-off.
- How the same ideas look in TypeScript with [Effection](https://frontside.com/effection/), where `yield*` plays the part of `yield from` and promises get wrapped at the edges
- The odd-looking `self.wind`, `self.tock`, and bare `yield` at the top of KERIpy generators, which exist so priming a generator doesn't run business logic

One practical warning from the talk: LLMs struggle with this model. Not knowing that HIO keeps its own time cost me somewhere between 40 and 80 dollars of Codex tokens on a single bug. Knowing how the Doist works lets you tell the model what it's missing.

If you'd rather start higher up the stack, my [KERIA tutorial](https://keri.foundation/confs/KERI/2026/apr/#keria-tutorial-kent-bull) from the day before covers the Signify and KERIA architecture built on top of this runtime.
