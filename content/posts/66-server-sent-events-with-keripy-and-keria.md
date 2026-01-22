+++
draft = true
title = "KERI Internals: Server-Sent Events with KERIpy and KERIA"
slug = "server-sent-events-with-keripy-and-keria"
date = "2026-01-08"

[taxonomies]
tags=["keri", "keria", "hio", "python", "sse", "server-sent-events", "wsgi", "falcon", "streaming", "http"]

[extra]
comment = true
+++

# KERI Internals: Server-Sent Events with KERIpy and KERIA

How does KERIpy stream mailbox messages to clients? This article explains how `QryRpyMailboxIterable` implements Server-Sent Events (SSE) over WSGI using Falcon's `rep.stream` interface—the only viable approach given Hio's architecture.

**Disclaimer**: This article is for a technical audience with intermediate to advanced programming knowledge. Familiarity with HTTP, Python iterators, and WSGI concepts is assumed.

## Introduction: The Streaming Problem

KERI agents need to receive messages asynchronously from their mailboxes. When a witness or other agent deposits a message in your mailbox, you need a way to learn about it without constantly polling. Server-Sent Events (SSE) provide an elegant HTTP-based solution: the server keeps the connection open and pushes events as they arrive.

But there's a challenge: KERIpy and KERIA use Hio, a cooperative multitasking async framework that operates over WSGI (Web Server Gateway Interface)—not ASGI (Asynchronous Server Gateway Interface). WSGI is synchronous by design. So how do you stream data in a synchronous protocol?

The answer lies in WSGI's iterator pattern and Falcon's `rep.stream` interface.

## Server-Sent Events: A Quick Primer

SSE is a W3C standard for servers pushing events to clients over HTTP. Unlike WebSockets, SSE is unidirectional (server to client) and works over standard HTTP. Each event has a simple text format:

```
id: 0
event: /receipt
retry: 5000
data: {"i": "EIaGMMWJFPmtXznY1IIiKDIrg-vIyge6mBl2QV8dDjI3", "t": "rct"}

```

Key fields:
- `id`: Event identifier (for reconnection)
- `event`: Event type/topic
- `retry`: Client reconnection timeout in milliseconds
- `data`: The actual payload

Events are separated by double newlines (`\n\n`). The `Content-Type` header must be `text/event-stream`.

## WSGI Streaming: The Iterator Pattern

WSGI (PEP 3333) allows response bodies to be iterables. Instead of returning a complete response, you return an iterator that the server calls `next()` on repeatedly:

```python
def wsgi_app(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/event-stream')])
    return MyIterator()  # Server calls next() repeatedly
```

This is the **only** way to stream data in WSGI. There's no `async/await`, no WebSocket upgrade, no bidirectional communication—just an iterator that yields chunks.

Hio's WSGI server (`Responder` class) specifically detects `text/event-stream` content type and sets special handling:

```python
# From hio/src/hio/core/http/serving.py
if u'content-type' in self.headers:
    if self.headers['content-type'].startswith('text/event-stream'):
        self.evented = True
```

The `Responder.service()` method then iterates over the application's response:

```python
def service(self):
    if not self.closed and not self.ended:
        if self.iterator is None:
            self.iterator = iter(self.app(self.environ, start_response=self.start))
        try:
            msg = next(self.iterator)
        except StopIteration:
            self.write(b'')  # Empty chunk terminates
            self.ended = True
        else:
            if msg:  # Only write if not empty—allows async processing
                self.write(msg)
```

The key insight: **yielding empty bytes `b''` keeps the connection alive without sending data**, allowing cooperative multitasking while waiting for messages.

## The QryRpyMailboxIterable Implementation

Located in `keri/app/indirecting.py`, `QryRpyMailboxIterable` is a two-phase iterator that handles streaming mailbox messages via SSE.

### Phase 1: Waiting for Authorization

When a client sends a mailbox query (`/mbx` route), the HTTP endpoint creates the iterable immediately:

```python
# From keri/app/indirecting.py - HttpEnd.on_post()
elif ilk in (Ilks.qry,):
    if sadder.ked["r"] in ("mbx",):
        rep.set_header('Content-Type', "text/event-stream")
        rep.status = falcon.HTTP_200
        rep.stream = QryRpyMailboxIterable(mbx=self.mbx, cues=self.qrycues, said=sadder.said)
```

But the iterator can't start streaming until the query is validated. It waits for a "stream" cue with a matching SAID (Self-Addressing Identifier):

```python
class QryRpyMailboxIterable:
    def __init__(self, cues, mbx, said, retry=5000):
        self.mbx = mbx
        self.retry = retry
        self.cues = cues
        self.said = said
        self.iter = None  # No MailboxIterable yet

    def __iter__(self):
        return self

    def __next__(self):
        if self.iter is None:
            if self.cues:
                cue = self.cues.pull()
                serder = cue["serder"]
                if serder.said == self.said:
                    kin = cue["kin"]
                    if kin == "stream":
                        # Authorization received—create the real iterator
                        self.iter = iter(MailboxIterable(
                            mbx=self.mbx, 
                            pre=cue["pre"], 
                            topics=cue["topics"],
                            retry=self.retry
                        ))
                else:
                    self.cues.append(cue)  # Wrong SAID, put it back

            return b''  # Keep connection alive while waiting

        return next(self.iter)  # Delegate to MailboxIterable
```

During this phase, every call to `__next__()` returns `b''` (empty bytes). Hio's server ignores empty chunks, keeping the HTTP connection alive without sending data. Meanwhile, other Hio doers process the query and eventually push a "stream" cue with the matching SAID.

### Phase 2: Streaming Mailbox Events

Once authorized, `QryRpyMailboxIterable` delegates to `MailboxIterable`, which does the actual work:

```python
class MailboxIterable:
    TimeoutMBX = 30000000  # ~347 days in seconds

    def __init__(self, mbx, pre, topics, retry=5000):
        self.mbx = mbx
        self.pre = pre
        self.topics = topics  # Dict of topic -> starting index
        self.retry = retry

    def __iter__(self):
        self.start = self.end = time.perf_counter()
        return self

    def __next__(self):
        if self.end - self.start < self.TimeoutMBX:
            if self.start == self.end:
                self.end = time.perf_counter()
                return bytearray(f"retry: {self.retry}\n\n".encode("utf-8"))

            data = bytearray()
            for topic, idx in self.topics.items():
                key = self.pre + topic
                for fn, _, msg in self.mbx.cloneTopicIter(key, idx):
                    data.extend(bytearray(
                        "id: {}\nevent: {}\nretry: {}\ndata: ".format(fn, topic, self.retry)
                        .encode("utf-8")
                    ))
                    data.extend(msg)
                    data.extend(b'\n\n')
                    idx = idx + 1
                    self.start = time.perf_counter()

                self.topics[topic] = idx
            
            self.end = time.perf_counter()
            return data

        raise StopIteration  # Timeout reached
```

**Key behaviors:**

1. **First iteration**: Sends `retry: 5000\n\n` to set client reconnection timeout
2. **Subsequent iterations**: Iterates through all topics, formats each message as an SSE event
3. **Batching**: Multiple messages available at once are concatenated into a single chunk
4. **Timeout**: Connection closes after ~347 days of inactivity (effectively infinite)
5. **Index tracking**: Remembers the last message index per topic to avoid duplicates

### SSE Event Format

Each mailbox message becomes a properly formatted SSE event:

```
id: 0
event: /receipt
retry: 5000
data: {"i": "EIaGMMWJFPmtXznY1IIiKDIrg-vIyge6mBl2QV8dDjI3", "t": "rct"}

```

The test confirms this format:

```python
# From tests/app/test_indirecting.py
msg = dict(i=hab.pre, t="rct")
mbx.storeMsg(topic=f"{hab.pre}/receipt", msg=json.dumps(msg).encode("utf-8"))
val = next(mbi)
assert val == (b'id: 0\nevent: /receipt\nretry: 1000\ndata: '
               b'{"i": "EIaGMMWJFPmtXznY1IIiKDIrg-vIyge6mBl2QV8dDjI3", '
               b'"t": "rct"}\n\n')
```

## Why This Is the Only (and Best) Approach

Phil Feairheller (author of this code) and Sam Smith (inventor of KERIpy) are correct: **`rep.stream` with an iterable is the only way to implement SSE over WSGI with Hio.**

Here's why:

### WSGI Constraints

1. **Synchronous by design**: WSGI has no concept of `async/await` or coroutines
2. **No WebSocket support**: WSGI cannot upgrade connections to bidirectional protocols
3. **Iterator pattern is the standard**: PEP 3333 defines iterables as the way to stream responses

### Why Not Alternatives?

| Approach | Why It Won't Work |
|----------|-------------------|
| **ASGI** | Would require rewriting Hio's entire HTTP stack |
| **WebSockets** | Not supported by WSGI; would need ASGI or raw sockets |
| **Long polling** | Works but is inefficient (new connection per message) |
| **Raw sockets** | Bypasses HTTP entirely, loses Falcon routing benefits |

### The Elegance of This Design

The iterator pattern actually fits Hio's cooperative multitasking model perfectly:

1. **Yielding `b''` enables cooperation**: Empty yields don't block; other doers can run
2. **No thread blocking**: Unlike synchronous I/O, the iterator yields control
3. **Natural backpressure**: If the client is slow, `next()` calls slow down
4. **Clean resource management**: `StopIteration` signals clean connection termination

## Similar Pattern: SignalIterable

KERIA uses the same pattern for its signals endpoint (`SignalIterable` in `keri/app/signaling.py`):

```python
class SignalIterable:
    TimeoutMBX = 300  # 5 minutes

    def __init__(self, signals, retry=5000):
        self.signals = signals
        self.retry = retry

    def __next__(self):
        if self.end - self.start < self.TimeoutMBX:
            if self.start == self.end:
                return bytes(f"retry: {self.retry}\n\n".encode("utf-8"))

            data = bytearray()
            while self.signals:
                sig = self.signals.popleft()
                topic = sig.topic
                if topic is not None:
                    data.extend(bytearray(
                        "id: {}\nretry: {}\nevent: {}\ndata: ".format(
                            sig.rid, self.retry, topic
                        ).encode("utf-8")
                    ))
                else:
                    data.extend(bytearray(
                        "id: {}\nretry: {}\ndata: ".format(
                            sig.id, self.retry
                        ).encode("utf-8")
                    ))
                data.extend(sig.raw)
                data.extend(b'\n\n')
            
            return bytes(data)

        raise StopIteration
```

The pattern is consistent: an iterator that yields SSE-formatted events, with empty yields when waiting for data.

## Summary

**What `QryRpyMailboxIterable` does:**
- Implements a two-phase iterator: authorization waiting, then message streaming
- Formats mailbox messages as SSE events (with `id`, `event`, `retry`, `data` fields)
- Batches multiple available messages into single chunks for efficiency
- Yields empty bytes during waiting phases to keep the connection alive

**Why this approach is used:**
- WSGI's iterator pattern is the **only** way to stream responses
- Falcon's `rep.stream` passes iterables directly to Hio's WSGI server
- Hio detects `text/event-stream` content type and handles it appropriately
- Empty yields enable Hio's cooperative multitasking to service other requests

**Why there's no alternative:**
- WSGI doesn't support async/await, WebSockets, or bidirectional communication
- Switching to ASGI would require rewriting Hio's entire HTTP layer
- The iterator pattern is elegant and fits Hio's cooperative model

When you need to stream data over HTTP in a WSGI environment, this is the way.

## References

1. WHATWG, "Server-sent events," HTML Living Standard. [https://html.spec.whatwg.org/multipage/server-sent-events.html](https://html.spec.whatwg.org/multipage/server-sent-events.html)

2. P. J. Eby, "PEP 3333 – Python Web Server Gateway Interface v1.0.1," Python Enhancement Proposals, Sep. 2010. [https://peps.python.org/pep-3333/](https://peps.python.org/pep-3333/)

3. S. Smith, "hio - Hierarchical Asynchronous Coroutines and I/O in Python," GitHub. [https://github.com/ioflo/hio](https://github.com/ioflo/hio)

4. WebOfTrust, "KERIpy - Python Implementation of KERI," GitHub. [https://github.com/WebOfTrust/keripy](https://github.com/WebOfTrust/keripy)

5. Falcon Contributors, "Falcon Web Framework," [https://falcon.readthedocs.io/](https://falcon.readthedocs.io/)

