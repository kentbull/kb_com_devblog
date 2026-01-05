+++
title = "Reading Mind Map: Key Event Receipt Infrastructure – KERI"
slug = "keri-reading-mind-map"
date = "2022-06-05"

[taxonomies]
tags=["keri", "acdc", "said", "decentralized-identity", "ssi"]

[extra]
comment = true
+++

# Reading Mind Map for KERI

> “Make things as simple as possible, but no simpler.”\
> – Albert Einstein.

In the spirit of keeping things simple for the new trust layer for the internet Dr. Samuel A Smith used the principle of “minimally sufficient means” to create Key Event Receipt Infrastructure, or KERI.

Welcome to KERI, the world of scalable, decentralized, authentic data
Allow me to give you a warm welcome to the world of Key Event Receipt Infrastructure, a special place in the self sovereign identity (SSI) ecosystem. Abbreviated as KERI, this flavor of decentralized key management infrastructure (DKMI) powers the main thrust of the movement for individual, personal control of data and privacy in the new private internet.

## Give me a map, please!

Say you want to learn KERI, what is your first step? Reading the white paper, all 141 pages of it? As of today the answer is yes. Yet the KERI white paper is only the beginning, not the end.

How can you come up to speed on all of the important IETF Draft Specifications around and important to KERI? Use the following pie graph and chart as a map:

{% image(path="/images/posts/05-keri-mind-map-01.webp", class="diagram",
    op="fit_width",
    alt="KERI Reading Mind Map Chart") 
    %}
KERI Reading Mind Map
{% end %}

## Guided Tour
1. The KERI whitepaper, the KERI IETF Draft spec, and the did:keri spec make up 51% of the required reading yet read the whitepaper first and then come back to read the KERI IETF Draft spec at the end after all of the other reading since the KERI IETF Draft spec includes items from the ACDC and CESR specs.
2. Then SAID and OOBI are the next good unit. SAID is a foundational concept for both the OOBI and ACDC specs. SAIDs are introduced in the KERI whitepaper though are important enough to have their own specification.
3. ACDC, PTEL, and IPEX are good to do next. PTEL and IPEX directly depend on ACDC to make sense.
4. Finish it off with CESR and CESR Proof Sigs, the text and binary communication protocol to make all of this high performance.


### KERIOX (Rust Implementation of KERI)

To learn about the Rust implementation of KERI using The Human Colossus’ repository you can review the following Rust source code directories (Rust modules) in order:

1. keriox_core/src/derivation
2. keriox_core/src/prefix
3. keriox_core/src/event
4. keriox_core/src/event_module
5. keriox_core/src/processor

## Why so many pages?

Print and select “Save as PDF” for each of the IETF Draft Specifications located at the table in the Web Of Trust KERI repository on GitHub as well as the KERI Whitepaper is where all the page counts located here came from. If you want to master KERI then roll up your sleeves and dig in. Whistle while you read!

## More Resources

All things KERI may be found on the KERI One website where you will also find the bio of the following character, Dr. Samuel A. Smith, the creator and inventor of KERI.

{% character(name="sam") %}
Hello! I'm Sam!
{% end %}

To meet Dr. Smith in person, and many other illustrious members of the SSI community join us at the next, as of the time of the first writing of this article, Internet Identity Workshop this November 15th to the 17th.


## IIW
This conference is where the brilliant minds and members of the SSI community come together every six months to share their knowledge and brag about their achievements. See you there!