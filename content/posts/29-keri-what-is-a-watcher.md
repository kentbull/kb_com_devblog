+++
draft = true
title = "KERI Concepts Series: What is a watcher"
slug = "keri-what-is-a-watcher"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERI Concepts Series: What is a watcher

In direct mode every controller is also a watcher of any controller it directly interacts with it because it verifies controller KELs and events. Direct mode is actually the first mode that was implemented in KERIpy. And the KERIpy codebase grew around using much of the direct mode verification functionality for everything in order to get a product out quickly for GLEIF, yet there are corner cases for indirect mode that are not addressed by direct mode. The direct mode code is not generalizable thus the need to

