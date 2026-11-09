+++
title = "Talk: KERIA Tutorial"
slug = "keria-tutorial"
date = "2026-11-09"

[taxonomies]
tags=["keri", "keria", "signify", "acdc", "talks", "conference"]

[extra]
comment = true
+++

The recording of my KERIA tutorial from the [KERI Conference, April 2026](https://keri.foundation/confs/KERI/2026/apr/) is up on keri.foundation:

**[KERIA Tutorial](https://keri.foundation/confs/KERI/2026/apr/#keria-tutorial-kent-bull)** (51 minutes, with video, [slides](https://keri.foundation/confs/KERI/2026/apr/slides/Kent%20Bull%20%EF%BD%9C%20KERIA%20Tutorial%20%EF%BD%9C%20KERI%20Conference%202026.pdf), and a [full transcript](https://keri.foundation/confs/KERI/2026/apr/transcripts/html/talks/keria-tutorial-kent-bull.html)).

<!-- more -->

## The short version

KERIA is the server half of the signing-at-the-edge architecture. A Signify client, in TypeScript or Python, holds the passcode and does all of the signing. A KERIA agent stores your key event logs, ACDCs, contacts, and OOBIs, and acts as a mailbox so messages still reach you when your device is offline. I call this a lightweight wallet, because all the user holds is an edge signing client. A heavyweight wallet like Locksmith keeps the keys and the data together in one local app, with no server agent.

The first question every team asks is where the keys are. The key material stored on the agent is always encrypted, using a key derived from a passcode that never leaves the edge. The client pulls the encrypted salts down, decrypts them in memory, and signs there. Someone who compromises the server, or who administers a multi-tenant agency, still can't use your keys.

## What the talk covers

- The boot and connect flow, which sets up a strict KERI delegation between the Signify controller and its agent
- How an agency hosts many agents, one per Signify controller, each with its own LMDB database
- Managed identifiers, which are the AIDs that actually issue credentials, and why you might want several
- A live demo in a developer wallet: connecting, creating a managed identifier, and reading its key event log as an annotated CESR stream
- Configuring a KERIA server, including which parts of the config every agent inherits

## Gotchas worth knowing up front

- Don't store the passcode in IndexedDB or localStorage, because any same-origin JavaScript can read it and exfiltrate it. On Ionic Capacitor, use the secure storage plugin.
- Set `KERI_AGENT_CORS=true` when you run `keria start`, or browser clients will fail on preflight requests.
- Passcodes are exactly 21 characters.
- The top-level `keria` property in the config file has to match the `--name` you pass to `keria start`, and it needs a `curls` value because that is where KERIA gets the port to bind.

If you want the configuration in detail, I wrote it up in [Configuring KERIpy Controllers, Witnesses, and KERIA Agents](@/posts/68-configuring-keripy-keria-controllers-witnesses.md). The day after this tutorial I went a level lower with [KERIpy Async Architecture](https://keri.foundation/confs/KERI/2026/apr/#keripy-async-architecture-kent-bull), on the HIO runtime underneath all of this.
