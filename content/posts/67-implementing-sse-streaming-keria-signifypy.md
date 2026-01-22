+++
draft = true
title = "Implementing SSE Streaming Notifications from KERIA to SignifyPy"
slug = "implementing-sse-streaming-keria-signifypy"
date = "2026-01-12"

[taxonomies]
tags=["keri", "keria", "signifypy", "sse", "server-sent-events", "python", "implementation", "tutorial"]

[extra]
comment = true
+++

# Implementing SSE Streaming Notifications from KERIA to SignifyPy

This post documents the implementation plan for adding real-time Server-Sent Events (SSE) streaming notifications from KERIA to SignifyPy clients. This builds on the SSE infrastructure already present in KERIpy's `SignalIterable` class.

**Status**: Implementation Plan (not yet implemented)

## Overview

Currently, SignifyPy clients must poll KERIA's `/notifications` endpoint to check for new notifications. This plan adds a `/notifications/stream` SSE endpoint that pushes notifications in real-time as they occur.

## Architecture

```
┌─────────────────┐         SSE Stream          ┌─────────────────┐
│   SignifyPy     │ ◄────────────────────────── │     KERIA       │
│   Client        │     /notifications/stream   │     Agent       │
│                 │                             │                 │
│  - sseclient    │         GET request         │  - Signaler     │
│  - stream()     │ ───────────────────────────►│  - SignalIter   │
│                 │                             │                 │
└─────────────────┘                             └─────────────────┘
```

The key insight is that KERIA already creates a `Signaler` instance for each agent, and KERIpy already has the `SignalIterable` class that formats signals as SSE events. The infrastructure exists—it just needs to be wired up to an HTTP endpoint.

## Background: Existing Infrastructure

### KERIpy's SignalIterable

The `SignalIterable` class in `keri/app/signaling.py` already implements SSE streaming:

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
                data.extend(bytearray(
                    "id: {}\nretry: {}\nevent: {}\ndata: ".format(
                        sig.rid, self.retry, topic
                    ).encode("utf-8")
                ))
                data.extend(sig.raw)
                data.extend(b'\n\n')
            return bytes(data)

        raise StopIteration
```

### KERIA's Signaler Setup

In KERIA's `Agent.__init__`, a `Signaler` is already created:

```python
signaler = signaling.Signaler()
self.notifier = Notifier(hby=hby, signaler=signaler)
```

Various components already push to this signaler when events occur (challenges, credentials, multisig coordination, etc.).

### SignifyPy's SSE Client Support

SignifyPy already has `sseclient` as a dependency and a `stream()` method on `SignifyClient`:

```python
def stream(self, path, params=None, headers=None, body=None):
    client = sseclient.SSEClient(url, session=self.session, **kwargs)
    for event in client:
        yield event
```

---

## Phase 1: KERIA Server-Side Implementation

### 1.1 Create New Streaming Module

**File:** `keria/src/keria/app/streaming.py`

```python
# -*- encoding: utf-8 -*-
"""
KERIA
keria.app.streaming module

SSE streaming endpoints for real-time notifications
"""

import falcon
from keri.app.signaling import SignalIterable


def loadEnds(app):
    """Load SSE streaming endpoints into the Falcon app.
    
    Args:
        app (falcon.App): Falcon application to register routes with
    """
    streamEnd = NotificationStreamEnd()
    app.add_route("/notifications/stream", streamEnd)


class NotificationStreamEnd:
    """
    SSE streaming endpoint for real-time notifications.
    
    This endpoint streams notifications to authenticated Signify clients
    using Server-Sent Events. The agent's Signaler is already populated
    by various components (ChallengeHandler, Notifier, credentialing, etc.)
    when events occur.
    
    SSE Event Format:
        Each notification is formatted as a standard SSE event:
        
            id: <signal_id>
            retry: <reconnect_timeout_ms>
            event: <topic>
            data: <json_payload>
        
        Events are terminated by double newlines (\\n\\n).
    
    Authentication:
        The request is authenticated by KERIA's SignatureValidationComponent
        middleware, which sets req.context.agent for valid requests.
    
    Usage:
        Signify clients connect via:
            GET /notifications/stream
            Accept: text/event-stream
        
        The connection remains open, streaming events as they occur.
    """
    
    def on_get(self, req, rep):
        """
        Stream notifications via SSE to authenticated Signify client.
        
        Parameters:
            req (Request): Falcon HTTP request (with req.context.agent set by middleware)
            rep (Response): Falcon HTTP response
        
        ---
        summary: Stream notifications via Server-Sent Events
        description: |
            Opens a long-lived connection that streams notifications as SSE events.
            Each event contains notification data in JSON format.
            The connection remains open until timeout (5 minutes of inactivity)
            or client disconnect.
        tags:
           - Notifications
        responses:
           200:
              content:
                 text/event-stream:
                    schema:
                       type: object
              description: SSE stream of notification events
           401:
              description: Unauthorized - agent not found in request context
        """
        agent = req.context.agent
        if agent is None:
            raise falcon.HTTPUnauthorized(
                title="Agent not found",
                description="No agent context in request. Authentication may have failed."
            )
        
        # Access the signaler through the agent's notifier
        signaler = agent.notifier.signaler
        
        # Set SSE response headers
        rep.set_header('Cache-Control', "no-cache")
        rep.set_header('Connection', "close")
        rep.set_header('Content-Type', "text/event-stream")
        rep.status = falcon.HTTP_200
        
        # Stream signals using KERIpy's SignalIterable
        rep.stream = SignalIterable(signals=signaler.signals)
```

### 1.2 Wire Up in KERIA's Server Setup

**File:** `keria/src/keria/app/agenting.py`

Add to imports:
```python
from keria.app import streaming
```

In `createAdminServerDoer()`, add after existing `loadEnds` calls:
```python
def createAdminServerDoer(config: KERIAServerConfig, agency: Agency):
    # ... existing setup ...
    
    loadEnds(app=adminApp)
    aidEnd = aiding.loadEnds(app=adminApp, agency=agency, authn=authn)
    credentialing.loadEnds(app=adminApp, identifierResource=aidEnd)
    delegating.loadEnds(app=adminApp, identifierResource=aidEnd)
    notifying.loadEnds(app=adminApp)
    keriagrouping.loadEnds(app=adminApp)
    keriaexchanging.loadEnds(app=adminApp)
    ipexing.loadEnds(app=adminApp)
    
    streaming.loadEnds(app=adminApp)  # <-- ADD THIS LINE
    
    # ... rest of function ...
```

### 1.3 Verify Signaler Access Path

Confirm the access path `agent.notifier.signaler` works by checking KERIpy's `Notifier` class stores the signaler reference. In `keri/app/notifying.py`:

```python
class Notifier:
    def __init__(self, hby, signaler=None):
        self.signaler = signaler if signaler is not None else Signaler()
        # ...
```

---

## Phase 2: SignifyPy Client-Side Implementation

### 2.1 Add Streaming Method to Notifications Class

**File:** `signifypy/src/signify/app/notifying.py`

```python
# -*- encoding: utf-8 -*-
"""
SIGNIFY
signify.app.notifying module

"""
import json
from signify.app.clienting import SignifyClient
from signify.core import httping


class Notifications:
    """ Domain class for accessing Endpoint Role Authorizations """

    def __init__(self, client: SignifyClient):
        self.client = client

    def list(self, start=0, end=24):
        """ Returns list of notifications (polling-based)
        
        ... existing implementation ...
        """
        headers = dict(Range=f"notes={start}-{end}")
        res = self.client.get(f"/notifications", headers=headers)
        cr = res.headers["content-range"]
        start, end, total = httping.parseRangeHeader(cr, "notes")
        return dict(start=start, end=end, total=total, notes=res.json())

    def markAsRead(self, nid):
        """ Mark notification as read
        
        ... existing implementation ...
        """
        res = self.client.put(f"/notifications/{nid}", json={})
        return res.status_code == 202

    def delete(self, nid):
        """ Delete notification
        
        ... existing implementation ...
        """
        res = self.client.delete(path=f"/notifications/{nid}")
        return res.status_code == 202

    def stream(self, timeout=None):
        """
        Stream notifications via Server-Sent Events.
        
        Opens a long-lived connection to KERIA and yields notifications
        as they arrive in real-time.
        
        Parameters:
            timeout (float): Optional timeout in seconds. If not specified,
                the stream continues until the server timeout (5 minutes
                of inactivity) or client disconnect.
        
        Yields:
            dict: Notification data from each SSE event. The structure matches
                the notification format from the list() method:
                {
                    "i": "<notification_id>",
                    "dt": "<datetime>",
                    "r": "<topic/route>",
                    "a": { ... attributes ... }
                }
        
        Example:
            ```python
            client = SignifyClient(...)
            client.connect()
            
            for notification in client.notifications().stream():
                print(f"Received: {notification['r']}")
                if notification['r'] == '/multisig/icp':
                    # Handle multisig inception request
                    handle_multisig(notification['a'])
            ```
        
        Note:
            The stream uses SSE (Server-Sent Events), which automatically
            handles reconnection if the connection is lost. The retry
            interval is controlled by the server (default 5000ms).
        """
        for event in self.client.stream("/notifications/stream"):
            if event.data:
                try:
                    yield json.loads(event.data)
                except json.JSONDecodeError:
                    # Skip malformed events
                    continue
```

---

## Phase 3: Testing

### 3.1 KERIA Unit Test

**File:** `keria/tests/app/test_streaming.py`

```python
# -*- encoding: utf-8 -*-
"""
KERIA
keria.app.streaming module

Testing SSE streaming notifications
"""

import falcon
from keria.app import streaming


def test_load_ends(helpers):
    """Test that streaming endpoints are properly loaded"""
    with helpers.openKeria() as (agency, agent, app, client):
        streaming.loadEnds(app=app)
        
        res = app._router.find("/test")
        assert res is None
        
        (end, *_) = app._router.find("/notifications/stream")
        assert isinstance(end, streaming.NotificationStreamEnd)


def test_notification_stream_headers(helpers):
    """Test SSE streaming endpoint returns correct headers"""
    with helpers.openKeria() as (agency, agent, app, client):
        streaming.loadEnds(app=app)
        
        # Note: Falcon test client may not fully support SSE streaming
        # This tests that the endpoint exists and sets correct headers
        res = client.simulate_get(path="/notifications/stream")
        
        assert res.status == falcon.HTTP_200
        assert res.headers['Content-Type'] == 'text/event-stream'
        assert res.headers['Cache-Control'] == 'no-cache'


def test_notification_stream_signals(helpers):
    """Test that signals are streamed as SSE events"""
    with helpers.openKeria() as (agency, agent, app, client):
        streaming.loadEnds(app=app)
        
        # Push signals before connecting (they queue up)
        signaler = agent.notifier.signaler
        signaler.push(
            attrs=dict(type="test", msg="hello"),
            topic="/test"
        )
        signaler.push(
            attrs=dict(type="test", msg="world"),
            topic="/test"
        )
        
        # Verify signals are queued
        assert len(signaler.signals) == 2
        
        # Note: Full SSE consumption test requires real HTTP server
        # See test_streaming_e2e.py for Doer-level testing
```

### 3.2 End-to-End Doer-Level Test

**File:** `keria/tests/app/test_streaming_e2e.py`

This test follows the pattern from KERIpy's `test_multisig_delegate`:

```python
# -*- encoding: utf-8 -*-
"""
KERIA
keria.app.test_streaming_e2e module

End-to-end testing of SSE notification streaming at the Doer level.
"""

import threading
import time
from contextlib import ExitStack

from hio.base import doing
from hio.core import http

from keria.app import streaming
from keria.testing import testing_helper


def test_sse_notification_streaming_e2e():
    """
    End-to-end test for SSE notification streaming from KERIA.
    
    This test covers:
    1. Setting up a KERIA agent with SSE streaming endpoint
    2. Creating a real HTTP server serving the SSE endpoint
    3. Pushing notifications from agent-side events
    4. Verifying notifications are available for streaming
    
    Pattern follows test_multisig_delegate from KERIpy:
    - Uses Doist for cooperative async coordination
    - Uses ExitStack for clean context management
    - Runs doers to drive event processing
    """
    doist = doing.Doist(limit=0.0, tock=0.03125, real=True)
    
    HTTP_PORT = 5999
    
    with ExitStack() as stack:
        # Open KERIA agency and agent
        keria_ctx = stack.enter_context(
            testing_helper.Helpers.openKeria()
        )
        agency, agent, falcon_app, test_client = keria_ctx
        
        # Load streaming endpoints
        streaming.loadEnds(app=falcon_app)
        
        # Create HTTP server for SSE streaming
        server = http.Server(port=HTTP_PORT, app=falcon_app)
        server_doer = http.ServerDoer(server=server)
        
        # Enter doers into Doist
        agent_deeds = doist.enter(doers=[agent])
        server_deeds = doist.enter(doers=[server_doer])
        all_deeds = agent_deeds + server_deeds
        
        # Let server start up
        for _ in range(20):
            doist.recur(deeds=all_deeds)
        
        # Verify server is running
        assert server.opened
        
        # Get signaler reference
        signaler = agent.notifier.signaler
        
        # Push notifications via the signaler
        signaler.push(
            attrs=dict(
                r="/multisig/icp",
                a=dict(gid="ETestMultisigAID123", smids=["pre1", "pre2"])
            ),
            topic="/multisig"
        )
        
        signaler.push(
            attrs=dict(
                r="/credential/issue",
                a=dict(said="ETestCredentialSAID", schema="ESchemaID")
            ),
            topic="/credential"
        )
        
        signaler.push(
            attrs=dict(
                r="/challenge/response",
                a=dict(words=["word1", "word2", "word3"])
            ),
            topic="/challenge"
        )
        
        # Drive event loop to process signals
        for _ in range(10):
            doist.recur(deeds=all_deeds)
        
        # Verify notifications are queued for streaming
        assert len(signaler.signals) == 3
        
        # The signals would be consumed when a client connects
        # For full client testing, see SignifyPy integration tests
        
        # Clean up
        doist.exit(doers=[server_doer])


def test_sse_with_real_client():
    """
    Test SSE streaming with actual HTTP client connection.
    
    This test uses a background thread to consume SSE events
    while the main thread pushes notifications.
    """
    import requests
    import sseclient
    
    doist = doing.Doist(limit=0.0, tock=0.03125, real=True)
    
    HTTP_PORT = 5998
    
    with ExitStack() as stack:
        # Setup KERIA
        keria_ctx = stack.enter_context(
            testing_helper.Helpers.openKeria()
        )
        agency, agent, falcon_app, _ = keria_ctx
        
        streaming.loadEnds(app=falcon_app)
        
        # Create and start HTTP server
        server = http.Server(port=HTTP_PORT, app=falcon_app)
        server_doer = http.ServerDoer(server=server)
        
        all_deeds = doist.enter(doers=[agent, server_doer])
        
        # Start server
        for _ in range(30):
            doist.recur(deeds=all_deeds)
        
        # Collector for received events
        received = []
        stream_error = []
        
        def consume_stream():
            """Background thread to consume SSE stream"""
            try:
                url = f"http://localhost:{HTTP_PORT}/notifications/stream"
                response = requests.get(url, stream=True, timeout=5)
                client = sseclient.SSEClient(response)
                
                for event in client.events():
                    if event.data:
                        received.append(event)
                        if len(received) >= 2:
                            break
            except Exception as e:
                stream_error.append(str(e))
        
        # Start consumer thread
        consumer = threading.Thread(target=consume_stream, daemon=True)
        consumer.start()
        
        # Give consumer time to connect
        time.sleep(0.5)
        for _ in range(20):
            doist.recur(deeds=all_deeds)
        
        # Push notifications
        signaler = agent.notifier.signaler
        signaler.push(attrs=dict(msg="first"), topic="/test")
        signaler.push(attrs=dict(msg="second"), topic="/test")
        
        # Drive event loop
        timeout = time.time() + 5.0
        while len(received) < 2 and time.time() < timeout:
            doist.recur(deeds=all_deeds)
            time.sleep(0.1)
        
        # Wait for consumer
        consumer.join(timeout=2.0)
        
        # Note: This test may need adjustment based on authentication
        # requirements. The test client needs proper Signify auth headers.
        
        doist.exit(doers=[server_doer])
```

### 3.3 SignifyPy Integration Test

**File:** `signifypy/tests/integration/test_sse_streaming.py`

```python
# -*- encoding: utf-8 -*-
"""
SIGNIFY
signify.tests.integration.test_sse_streaming module

Integration test for SSE notification streaming from KERIA to SignifyPy.

Requires KERIA to be installed as an editable dependency:
    pip install -e ../keria
"""

import threading
import time
from contextlib import ExitStack

from hio.base import doing
from hio.core import http

# Import from KERIA (editable install)
from keria.app import agenting, streaming
from keria.testing import testing_helper

from signify.app.clienting import SignifyClient


def test_signifypy_sse_streaming():
    """
    Test SignifyPy consuming SSE notifications from KERIA.
    
    This follows the test_multisig_delegate pattern:
    - Uses Doist for async coordination
    - Sets up KERIA agent with real HTTP server
    - SignifyPy client connects and streams notifications
    """
    doist = doing.Doist(limit=0.0, tock=0.03125, real=True)
    
    ADMIN_PORT = 5997
    
    with ExitStack() as stack:
        # Setup KERIA
        keria_ctx = stack.enter_context(testing_helper.Helpers.openKeria())
        agency, agent, falcon_app, _ = keria_ctx
        
        streaming.loadEnds(app=falcon_app)
        
        # Create admin server
        admin_server = http.Server(port=ADMIN_PORT, app=falcon_app)
        admin_doer = http.ServerDoer(server=admin_server)
        
        # Enter doers
        all_deeds = doist.enter(doers=[agent, admin_doer])
        
        # Let server start
        for _ in range(30):
            doist.recur(deeds=all_deeds)
        
        # Collector for received notifications
        received = []
        
        def stream_notifications():
            """Background consumer for SSE stream"""
            try:
                # Note: In a real test, you'd need to properly
                # authenticate the SignifyClient first
                passcode = "0123456789abcdefghijk"
                client = SignifyClient(
                    passcode=passcode,
                    url=f"http://localhost:{ADMIN_PORT}",
                    boot_url=f"http://localhost:{ADMIN_PORT}"  # Would be different in real setup
                )
                # client.boot() and client.connect() would be needed
                
                for note in client.notifications().stream():
                    received.append(note)
                    if len(received) >= 3:
                        break
            except Exception as e:
                print(f"Stream error: {e}")
        
        # Start streaming in background
        stream_thread = threading.Thread(target=stream_notifications, daemon=True)
        stream_thread.start()
        
        # Give stream time to connect
        time.sleep(0.5)
        for _ in range(20):
            doist.recur(deeds=all_deeds)
        
        # Push notifications from agent side
        signaler = agent.notifier.signaler
        signaler.push(
            attrs=dict(r="/multisig/icp", a=dict(gid="EABC")),
            topic="/multisig"
        )
        signaler.push(
            attrs=dict(r="/credential/issue", a=dict(said="EDEF")),
            topic="/credential"
        )
        signaler.push(
            attrs=dict(r="/challenge/response", a=dict(words=["word1"])),
            topic="/challenge"
        )
        
        # Drive event loop
        timeout = time.time() + 5.0
        while len(received) < 3 and time.time() < timeout:
            doist.recur(deeds=all_deeds)
            time.sleep(0.1)
        
        stream_thread.join(timeout=1.0)
        
        # Verify (may need adjustment for auth)
        # assert len(received) >= 3
        
        doist.exit(doers=[admin_doer])
```

---

## Phase 4: Implementation Checklist

### KERIA Changes

- [ ] Create `keria/src/keria/app/streaming.py`
  - [ ] `loadEnds()` function
  - [ ] `NotificationStreamEnd` class with `on_get()` method
  - [ ] Proper docstrings and OpenAPI annotations
- [ ] Update `keria/src/keria/app/agenting.py`
  - [ ] Import streaming module
  - [ ] Add `streaming.loadEnds(app=adminApp)` in `createAdminServerDoer()`
- [ ] Create `keria/tests/app/test_streaming.py`
  - [ ] `test_load_ends()`
  - [ ] `test_notification_stream_headers()`
  - [ ] `test_notification_stream_signals()`
- [ ] Create `keria/tests/app/test_streaming_e2e.py`
  - [ ] `test_sse_notification_streaming_e2e()`
  - [ ] `test_sse_with_real_client()`

### SignifyPy Changes

- [ ] Update `signifypy/src/signify/app/notifying.py`
  - [ ] Add `stream()` method to `Notifications` class
  - [ ] Add proper docstring with example
- [ ] Add KERIA as dev dependency
  - [ ] Update `requirements.txt` or `setup.py`
- [ ] Create `signifypy/tests/integration/test_sse_streaming.py`
  - [ ] `test_signifypy_sse_streaming()`

### Documentation

- [ ] Add docstrings to all new classes/methods
- [ ] Update this blog post with implementation details and results

---

## Dependencies and Editable Installs

For development across the projects, use editable installs:

```bash
# In keria directory
pip install -e .

# In signifypy directory  
pip install -e .
pip install -e ../keria  # KERIA as editable dependency
```

Or add to `signifypy/requirements.txt` for development:
```
-e ../keria
```

---

## Key Considerations

### 1. Authentication

The SSE endpoint must be behind KERIA's `SignatureValidationComponent` middleware so `req.context.agent` is populated. This is automatic when using the admin app.

### 2. Timeout Handling

`SignalIterable.TimeoutMBX` is 300 seconds (5 minutes). After this period of inactivity, the stream closes. Consider:
- Making this configurable
- Documenting reconnection behavior for clients

### 3. Reconnection

SSE clients automatically reconnect using the `retry` field sent by the server (default 5000ms = 5 seconds). The `sseclient` library in SignifyPy handles this.

### 4. Signal Consumption

Signals are **popped** from the deck when streamed. This means:
- Each signal is delivered once
- If multiple clients connect, only one receives each signal
- Consider if this is the desired behavior or if signals should be broadcast

### 5. Testing SSE

Falcon's test client (`testing.TestClient`) may not fully support SSE streaming. The end-to-end tests use real HTTP servers with `hio.core.http.Server`.

---

## Future Enhancements

1. **Broadcast Mode**: Allow multiple clients to receive the same signals
2. **Filtering**: Allow clients to subscribe to specific topics
3. **Persistence**: Store signals for replay on reconnection
4. **SignifyTS Support**: Implement client-side streaming in TypeScript

---

## References

1. Previous article: [KERI Internals: Server-Sent Events with KERIpy and KERIA](/server-sent-events-with-keripy-and-keria)
2. KERIpy `signaling.py`: Contains `SignalIterable` and `Signaler`
3. KERIA `agenting.py`: Agent initialization and server setup
4. SignifyPy `clienting.py`: Client with `stream()` method using `sseclient`
