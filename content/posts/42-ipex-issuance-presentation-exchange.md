+++
draft = true
title = "DRAFT – KERI Spec Series: IPEX – Issuance and Presentation Exchange Protocol"
slug = "ipex-issuance-presentation-exchange"
date = "2026-01-04"

[taxonomies]
tags=["keri"]

[extra]
comment = true
+++

# DRAFT – KERI Spec Series: IPEX – Issuance and Presentation Exchange Protocol

Point:  [Last two](https://trustoverip.github.io/tswg-ipex-specification/draft-ssmith-ipex.html#section-3.1-4) paragraphs in section 3.1

## Explanation of Spec Language

3.1 discussion elaboration

> To elaborate, the SAID of a given variant is useful even when it is not the SAID of the variant the Issuer signed because during graduated disclosure the Discloser MAY choose to sign that given variant to fullfill a given step in an IPEX graduated disclosure transaction. The Discloser thereby can make a verifiable disclosure in a given step of the SAD of a given variant that fulfills a commitment made in a prior step via its signature on merely the SAID of the SAD of the variant so disclosed. IPEX Spec

To elaborate, the SAID of a given variant is useful even when it is not the SAID of the variant the Issuer signed because during graduated disclosure the Discloser MAY choose to sign that given variant to fullfill a given step in an IPEX graduated disclosure transaction. The Discloser thereby can make a verifiable disclosure in a given step of the SAD of a given variant that fulfills a commitment made in a prior step via its signature on merely the SAID of the SAD of the variant so disclosed.

Means a disclosee can trust a block of data is authentic and properly disclosed during an IPEX exchange by receiving a discloser’s signature on a SAID of that block.

> For example, in the bulk issuance of an ACDC, the Issuer only signs the blinded SAID of the SAD that is the Compact variant of the ACDC not the SAD itself. IPEX Spec

For example, in the bulk issuance of an ACDC, the Issuer only signs the blinded SAID of the SAD that is the Compact variant of the ACDC not the SAD itself.

“not the SAD itself” means “not the [compact variant] SAD itself”

## Visuals

Show a visual of a Merkle Tree and an ACDC Tree with nodes in various compacted or uncompacted views to express what both an uncompaction and compaction process look like.

## Workflow

Can you respond to an apply with a grant or do you always need to go apply->offer->agree->grant?

## Typos and Grammatical Mistakes

## Section 3.1

satisfied with either directly -> satisfied either directly

agaisnt -> against

### Share this:

- Click to share on X (Opens in new window)
				X
- Click to share on Facebook (Opens in new window)
				Facebook
- 

Categories: [Uncategorized](https://kentbull.com/category/uncategorized/)

Tagged as: [temp-publish](https://kentbull.com/tag/temp-publish/)

### Kent Bull
