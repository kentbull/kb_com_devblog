+++
draft = true
title = "TSP Open vs. Closed Mode"
slug = "tsp-open-vs-closed-mode"
date = "2025-04-22"

[taxonomies]
tags=["uncategorized"]

[extra]
comment = true
+++

# TSP Open vs. Closed Mode

Trust Spanning Protocol comes in two modes, closed mode and open mode. Why are there two and how are they different?Closed mode allows the KERI ecosystem to use only KERI AIDs and not worry about DIDs. It lacks interoperability with any non-KERI TSP implementations yet avoids the insecure parts of TSP.

Open mode provides interoperability to the KERI ecosystem through `did:webs` yet is not as secure as closed mode.

Attacks on these three correlation vectors are removed: time of departure, time of arrival, size of packet. All of these are enough to correlate packets even if using the TOR network.

Allow all packets to be padded to a uniform size. SPAC already has no metadata about the packet.

HPKE Sam considers experimental because they are not more vetted, such as making sure all loops are uniform in the code to protect against side channel attacks. You need an implementation that is written by a cryptographic programmer.

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
