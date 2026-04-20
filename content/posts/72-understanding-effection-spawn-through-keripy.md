+++
draft = true
title = "Understanding Effection's spawn Through a KERIpy and HIO Mental Model"
slug = "understanding-effection-spawn-through-keripy"
date = "2026-03-28"

[taxonomies]
tags=["keri", "keripy", "hio", "effection", "typescript", "concurrency"]

[extra]
comment = true
+++

# Understanding Effection's spawn Through a KERIpy and HIO Mental Model

If you come to `keri-ts` from KERIpy, one of the first runtime questions you hit is this:

```typescript
const runtimeTask = yield* spawn(function*() {
  yield* runAgentRuntime(runtime);
});
```

What exactly is `spawn()` doing here?

If your brain was trained on HIO and KERIpy, the easiest way to understand it is not to start with JavaScript promises. The right place to start is with the HIO runtime concepts you already know: `Doist`, `DoDoer`, `Doer`, `extend`, `remove`, and the idea of a long-lived runtime host that owns a set of child tasks.

This post explains how to translate that mental model into Effection.

The short version is:

- Effection `spawn()` is conceptually similar to `DoDoer.extend(...)`.
- It is not a new OS thread.
- It creates a child task in the current runtime tree.
- It returns an explicit handle to that child task.
- That child task is owned by the parent and cannot outlive it.

That last point is where Effection becomes more explicit and, in some ways, stricter than HIO.

## Why this matters for `keri-ts`

In KERIpy, when you read code like `kli incept`, `kli init`, a witness runtime, or a watcher runtime, you are usually reading code built around HIO scheduling concepts:

- the `Doist` is the root scheduler,
- `DoDoer` instances are hierarchical task containers,
- `Doer` instances are the individual tasks,
- and `self.extend([...])` dynamically adds child tasks into the active runtime.

In `keri-ts`, the runtime is built with [Effection](https://frontside.com/effection), not HIO. So the names change, but the underlying need does not. We still need:

- a root runtime host,
- long-lived background protocol processing,
- sibling activities like an HTTP server and a protocol runtime loop,
- explicit shutdown,
- predictable cleanup,
- and a way to keep runtime ownership obvious.

If you do not build a translator between these two models in your head, Effection code looks magical when it is not.

## The tight mapping table

This is the translation table I wish I had written down earlier:

| HIO / KERIpy | Effection / `keri-ts` | Mental model |
|---|---|---|
| `Doist` | `run(() => rootOperation)` | Root owner of the whole runtime tree |
| `DoDoer` | A long-lived parent `Operation` that can `spawn()` children | Nested runtime host or subtree owner |
| doer or `doing.doify(...)` | generator `Operation` | One unit of managed work |
| `self.extend([...])` | `const task = yield* spawn(...)` | Dynamically add a child to the current runtime tree |
| `self.remove([...])` | `yield* task.halt()` | Dynamically remove a child and wait for teardown |
| `doers` / `deeds` | child tasks inside the current Effection scope | The active scheduled children |
| `yield self.tock` | any suspension point, plus an explicit runtime yield seam | Give the scheduler a chance to run other work |
| `enter / exit / abort / close` | `try/finally`, cancellation, task halt, structured cleanup | Lifecycle and cleanup semantics |
| `always=True` on a host `DoDoer` | `while (true)` or another long-lived blocking operation | Keep the host alive so it can continue managing children |
| child completion | `yield* task` | Wait for child result or completion |

This mapping is not exact in every respect, but it is good enough to reason accurately about `keri-ts` runtime structure.

## The key analogy

The nearest HIO move to Effection `spawn()` is this kind of pattern in KERIpy:

```python
obi = Oobiery(hby=hby)
self.extend(obi.doers)

...

self.remove(obi.doers)
```

or this kind of pattern in `kli incept`:

```python
witDoer = WitnessReceiptor(hby=self.hby)
receiptor = Receiptor(hby=self.hby)
self.extend([witDoer, receiptor])
```

The conceptual move is the same:

1. create child work,
2. attach it to the currently running host,
3. let it execute while the parent continues managing other work,
4. later remove or stop it.

In Effection, the corresponding move is:

```typescript
const runtimeTask = yield* spawn(function*() {
  yield* runAgentRuntime(runtime);
});
```

and later:

```typescript
yield* runtimeTask.halt();
```

That is the same family of runtime design.

## Where the analogy is strong

The analogy is strong in three ways.

### 1. Both express hierarchical concurrency

HIO explicitly models task hierarchies with `Doist -> DoDoer -> Doer`.

Effection explicitly models task hierarchies with `run() -> parent task -> spawned child tasks`.

In both cases, the purpose is the same: organize runtime work as a tree, not as a pile of disconnected callbacks.

### 2. Both are cooperative, not magical

Neither HIO nor Effection gives you automatic parallelism in the sense of new CPU threads. Both are concurrency frameworks that rely on yielding and suspension boundaries.

That means the right mental question is:

> Where does this task yield so other work can run?

That question matters in both systems.

### 3. Both are about runtime ownership

In good HIO code, you should be able to answer:

- Which `DoDoer` owns this child?
- When is the child added?
- When is it removed?
- What happens if the parent exits?

In good Effection code, you should be able to answer:

- Which parent task spawned this child?
- Who holds the `Task` handle?
- When is `halt()` called?
- What happens if the parent task crashes or exits?

Those are the same architectural concerns.

## Where the analogy breaks

This is the important part. If you stop at "Effection `spawn()` is like `extend()`," you will understand the happy path but miss the sharper guarantees.

### Effection gives you a first-class child task handle

`self.extend([...])` mutates the scheduler state. It adds new doers into the active runtime set.

Effection `spawn()` does that kind of dynamic child attachment, but it also returns a first-class `Task` handle.

That handle can be:

- awaited,
- halted,
- and used as the explicit ownership token for the child runtime.

This is more explicit than the usual HIO `extend()` feel.

### Effection is more explicit about structured concurrency

Effection's docs emphasize that child tasks do not outlive their parent scope. That is a strong structured concurrency guarantee.

This means `spawn()` is not just "register some more work with the scheduler." It means:

- create a child task,
- attach it to the current parent task,
- propagate failures correctly,
- and guarantee shutdown with parent teardown.

In other words, `spawn()` is closer to:

> `extend(...)` plus a returned child-task handle plus stronger parent-child lifetime guarantees

than it is to bare `extend(...)`.

### Effection hides the scheduler internals more than HIO

One of HIO's strengths is that the scheduling machinery is right there in the open:

- `tyme`
- `tock`
- `deeds`
- `recur`
- `enter`
- `exit`

You can read the scheduler and understand what one cycle does.

Effection is less explicit at that level. It exposes higher-level concepts:

- `Operation`
- `Task`
- `Scope`
- `spawn`
- `run`
- `halt`

This is a different style. It is usually cleaner at the application layer, but it means you need a cleaner conceptual model or else everything looks like hidden runtime magic.

## `Operation` versus `Task`

This distinction is one of the most important differences to internalize.

An `Operation` in Effection is a description of work. By itself, it does not run.

A `Task` is a running instance of an operation.

This is why these two things are different:

```typescript
runAgentRuntime(runtime)
```

and

```typescript
yield* spawn(function*() {
  yield* runAgentRuntime(runtime);
});
```

The first is an operation expression. The second actually creates a child task and starts it under the current parent task.

If you come from HIO, one way to think about this is:

- an `Operation` is closer to a doer definition,
- a `Task` is closer to a live scheduled child that is currently part of the runtime tree.

That is not a perfect mapping, but it is directionally correct and useful.

## How the `agentCommand()` runtime is structured

The `tufa agent` command in `keri-ts` creates a shared runtime and then needs two long-lived activities to coexist:

1. the protocol runtime loop
2. the HTTP server

That is why it does this:

```typescript
const runtimeTask = yield* spawn(function*() {
  yield* runAgentRuntime(runtime);
});

try {
  yield* startServer(port, consoleLogger, runtime);
} finally {
  yield* runtimeTask.halt();
}
```

This is a very clean structured-concurrency shape.

The parent command task:

- creates the shared runtime,
- spawns the child runtime loop,
- blocks on the server lifetime,
- and then halts the child runtime during teardown.

This is exactly the kind of pattern that would be awkward or wrong without a child-task primitive.

## Why `spawn()` is appropriate here

It is appropriate because the runtime loop and the server are both long-lived.

If the code were written like this:

```typescript
yield* runAgentRuntime(runtime);
yield* startServer(port, consoleLogger, runtime);
```

the server would never start, because `runAgentRuntime()` is intentionally continuous.

So we need concurrency.

The question is not whether concurrency is needed. It is what kind of concurrency abstraction is the right fit.

Effection `spawn()` is the right fit because it gives us:

- child concurrency,
- explicit ownership,
- explicit shutdown,
- failure propagation,
- and a handle that keeps runtime lifetime visible in the source code.

That is exactly what this kind of protocol host should want.

## `spawn()` is not a thread

This is one of the most common mistakes people make when they first read Effection code.

`spawn()` does not mean:

- new OS thread,
- new process,
- or magical background execution independent of the rest of the runtime.

It means:

- create a child task in the same cooperative runtime,
- let it make progress as the scheduler runs,
- and keep its lifecycle attached to the parent.

That is why scheduler fairness matters.

If a long-lived child task never yields in a useful way, it can starve sibling work even though the concurrency structure itself is correct.

## Why the runtime needs an explicit yield seam

This is where the HIO intuition helps again.

In HIO, you expect a task to yield through `yield self.tock` and re-enter on the next scheduler cycle. There is an obvious notion of giving control back to the scheduler.

In `keri-ts`, the shared runtime loop does the same thing conceptually through a function named `runtimeTurn()`.

The purpose of that function is not protocol logic. Its purpose is scheduler fairness. It gives the host a chance to run sibling tasks, timers, and I/O completions between runtime turns.

So the long-lived runtime looks conceptually like this:

```typescript
while (true) {
  yield* processRuntimeTurn(runtime);
  yield* runtimeTurn();
}
```

That is closer to the HIO intuition than it first appears:

- `processRuntimeTurn()` is one bounded pass of protocol work
- `runtimeTurn()` is the explicit "give the scheduler room to breathe" seam

That is not identical to `yield self.tock`, but it fills a similar conceptual role in the runtime design.

## The compact mental model

If you only remember one thing from this article, remember this:

> Effection `spawn()` is conceptually like `DoDoer.extend(...)`, but semantically it is stronger because it returns an explicit child task handle and enforces structured parent-child lifetime.

That is the translation layer.

From there, the rest gets much easier:

- `run()` is the root scheduler entrypoint, roughly the `Doist` role
- a long-lived command operation is roughly a `DoDoer`
- `spawn()` is how you dynamically attach child work
- `task.halt()` is how you explicitly tear child work down
- and explicit yield points matter because this is cooperative concurrency, not parallel execution

## Final takeaway

The wrong mental model is:

> Effection `spawn()` is just some JavaScript background task helper.

The better mental model is:

> Effection `spawn()` is the `keri-ts` way to express hierarchical child runtime ownership, much like `DoDoer.extend(...)` in KERIpy, but with a more explicit task handle and stronger structured concurrency guarantees.

Once you see that, the `keri-ts` runtime becomes much easier to read.

The point is not that HIO and Effection are the same. They are not. The point is that they solve a similar class of runtime-ownership problem, and if you already understand one, you can use it to learn the other much faster.
