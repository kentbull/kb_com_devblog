+++
draft = true
title = "KERI Internals Part 2: HIO Async Fundamentals"
slug = "keri-internals-hio-fundamentals"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERI Internals Part 2: HIO Async Fundamentals

What is Hio? Pronounced “hee-oh,” it is the asynchronous (async) coroutine and async IO framework built into many parts of the KERI ecosystem. A look into the code in the KERI space likely brought you into the Python reference implementation where you saw things like the Doer, DoDoer, and Doist classes along with their lifecycle functions like recur, enter, and exit. This post explains the async coroutine part of Hio, which is the foundation for everything else in Hio, and shows example usage along with a style guide so you can learn how to effectively use Hio. A [presentation on Hio](https://github.com/SmithSamuelM/Papers/blob/master/presentations/Hio.web.pdf) from its author, Dr. Samuel Smith, is another way yo ucan learn about hio.

Getting into it, Hio is what is called a weightless, hierarchical, asynchronous coroutine and I/O library built around the concept of flow based programming. It allows for deterministic coroutine scheduling (execution) which is very good for testability.

Define what functions need to be overidden when implementing a Doer and DoDoer and why.

Clearly define tock, tyme, tymth, timeouts, recur. When does a task terminate? How should data flow into and out of a task occur?

What happens when a Doer wakes up? What is the lifecycle of a Doer, DoDoer, and Doist?

Examples:

- Show examples that are not related to KERI

putting together KERIpy with HIO makes it easy to get lost

Escrow = Queue

- Show debugging through multiple components within one integration test
- I see three components, what happens if I add a fourth one?

Code Read Through:

Multiple Inheritance

- where is it used and why?

hioing.Mixin

subclasses

hio.base

filing.Filer
- tyming.Tymist
- tyming.Tymee

- memoing.Memoer
- udping.Peer

- naming.Namer
- timing.Timer

HIO Doer Lifecycle

- 

Cycle Time

- what does one tick of the clock look like? Where is time expressed in the system?

Outline:

- Lifecycle of Doist

show time start and time ticking
- doist.do

visual diagram
- code walkthrough

enter
- timer start
- recur
- exit

close

- DoDoer.do

visual diagram
- code walkthrough

__init__

resource init

- advances subtasks to first yield (primes their generators)

sets Doer tymth

- resource cleanup

- Doer() – __call__ -> Doer.do

Appendix:

What other capabilities does HIO provide?

- File IO
- Serial Console IO
- TCP
- HTTP
- Unix Domain Sockets
- UDP and reliable UDP
- Data Structures & Collections

Deck: double ended queue
- Hict: case insensitive multi-value dict, insertion ordered
- Mict: multi-value dict, insertion ordered

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
