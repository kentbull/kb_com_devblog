+++
draft = true
title = "KERIpy Technical Series: Delegation Ceremony Change from 1.1.x to 1.2.x"
slug = "keripy-delegation-ceremony-change"
date = "2025-04-22"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# KERIpy Technical Series: Delegation Ceremony Change from 1.1.x to 1.2.x

The delegated inception ceremony changed between versions 1.1.x of KERIpy and 1.2.x of KERIpy affecting both single signature identifiers and multisignature identifiers. Here’s what you need to know.

TODOs:

- Get the test scripts working with KERIpy 1.2.x keystores. Why doesn’t this work? What is missing?
- Diagram the old ceremony and the new ceremony. Show the place where an attacker can harm things in the old ceremony.
- Make a list that clarifies which old versions of KERIpy, KERIA, and SignifyTS are not compatible with the 1.2.x releases. Get specific Git SHAs and dates so we know the precise cutoff of when things became backwards incompatible.
- Precisely define what is backwards incompatible, specifically that old witnesses must be replaced with new ones. Show the migration path of witnesses with `kli migrate` and any other migration work.

Summary

- Old Ceremony

The **delegatee** creates a delegated inception event (type: `dip`), single signature or multisignature, and declares a **delegator** prefix. This causes the **delegatee** to send the delegated inception event to the **delegator**.
- The **delegator** receives this delegation inception request and approves (`kli delegate confirm`) or denies the request. The **delegator** used to approve the delegation after receiving only the delegated inception event from the **delegatee** (with the **delegatee’s** signature) and before receiving any witness receipts from the **delegatee’s** witnesses.

<graphic of delegation including seal and signatures.

- This means old 1.1.x witnesses cannot perform the 1.2.x delegation ceremony because they will never get the **delegator’s** seal since the delegator in 1.2.x is waiting on receiving **delegatee** witness receipts prior to anchoring the delegation seal.

- **Delegatee** creates a delegated inception event, sends the event (`dip`) to its witnesses, and then sends both the delegated inception event and the witness receipts of that event to the **delegator**.
- **Delegator** receives both the delegated inception event and the delegatee’s witness receipts of that event.
- **Delegator** approves or denies the delegation. On approval the delegator creates the delegation seal and anchors it to its KEL in an interaction event.
- The **delegatee** then should refresh the keystate of the **delegator** to learn of the interaction event containing the delegation approval.

Updated details from Apr 22, 2025

The details were that the delegation ceremony significantly changed from the old to the new ceremony.

### Old KERIpy 1.1.x delegation ceremony

1. `DELEGATE` creates delegated inception (`dip`) event and immediately sends it to `DELEGATOR` **PRIOR** to getting witness receipts from the **DELEGATE WITNESSES**, which occurs at the end of the process.
2. `DELEGATOR` anchors the delegated inception (`dip`) event in an interaction event (`ixn`), the approval of the delegation
3. `DELEGATOR` gets receipts from the **DELEGATOR WITNESSES** of this approval and
4. `DELEGATOR` sends both the interaction event and its receipts back to the `DELEGATE`.
5. `DELEGATE` sends the `dip`, the `ixn`, and the `ixn` receipts to the **DELEGATE WITNESSES** after which those witnesses then receipt the `DELEGATE`‘s `dip` event.
6. Delegation is complete.

Witnesses with the OLD delegation ceremony will not accept the `dip` event to witness it until after they receive the delegation approval `ixn` and its receipts, meaning KERIpy 1.1.x witnesses WILL NOT WORK with KERIpy 1.2.x delegation since they will wait forever for the delegation approval `ixn` and its receipts yet those are not sent by the delegate in 1.2.x until after the delegate gets its `dip` receipts.

### New KERIpy 1.2.x+ delegation ceremony

In KERIpy 1.2.0+ the

1. `DELEGATE` creates the `dip` event
2. `DELEGATE` receipts the `dip` immediately with the **DELEGATE WITNESSES** and
3. `DELEGATE` sends both the `dip` and its receipts to the `DELEGATOR`.
4. `DELEGATOR` anchors the `dip` in an approval `exn`,
5. `DELEGATOR` receipts the approval `ixn` with the **DELEGATOR WITNESSES** and
6. `DELEGATOR` sends the approval `ixn` and its receipts back to the `delegate`.
7. Delegation is complete.

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
