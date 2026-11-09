+++
title = "KERIpy Async Architecture"
description = "The execution model behind KERIpy: structured concurrency, HIO's Doist, Doers, and DoDoers, and what you need to know to read and extend the code."
date = 2026-04-23

[extra]
video = { link = "https://keri.foundation/confs/KERI/2026/apr/#keripy-async-architecture-kent-bull", thumbnail = "images/talks/keripy-async-architecture.jpg" }

[extra.organizer]
name = "KERI Conference April 2026"
link = "https://keri.foundation/confs/KERI/2026/apr/"
+++

At the April 2026 KERI Conference I explained the runtime underneath KERIpy. The talk starts with why structured concurrency exists, then traces `kli incept` through HIO's scheduler and tasks so you have a working mental model of how KERIpy runs.
