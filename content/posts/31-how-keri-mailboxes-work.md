+++
draft = true
title = "How KERI Mailboxes work"
slug = "how-keri-mailboxes-work"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# How KERI Mailboxes work

The Mailboxer is a messaging component in the KERI core library that accepts messages on a topic for a given prefix using the “recipient/topic” topic namespacing strategy. It is typically used with a Respondant message router that routes any type of KERI message, using a Poster, to the appropriate Mailboxer instance.

