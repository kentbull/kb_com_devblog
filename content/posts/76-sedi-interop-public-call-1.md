+++
draft = true
title = "SEDI Interop Work Group - Call 1"
slug = "sedi-interop-call-1"
date = "2026-05-08"

[taxonomies]
tags=["sedi", "utah", "bramwell", "mcewan", "interop", "workgroup"]

[extra]
comment = true
+++

Today the first SEDI Interop Work Group call happened. 16 people were in attendance, some from the Harvard Media Lab, some from the State of Utah, some from the KERI community, some from the W3C community.

Christopher Bramwell - 
George McEwan - 
Philip Feairheller
Manu Sporny - standards at the W3C and California DMV deploying digital credentials for the state of California and the retail sector.
Alberto Leon - senior research engineer
Brandon Miller - principal engineer applied social media lab
Brent Zundel - Yubico, chair of W3C Credentials working group, OAuth working groups at IETF, etc., everywhere identity standards are
Deepanker Saxena - 
Dmitri Zagidulin - MIT, Co-chairing, co-editing many decentralized identity and credential specs, emphasis on interop
Eric Scouten - Principal Scientist and chair of Creator Assertions Working Group (CAWG)
Jim St. Clair - digital credentials in health care
Joe Jackson - CTO for State of Utah and division of technology services
Kent Bull - 
Samuel Smith - 
Thomas Mayfield - Cardano and Veridian
Will Seggos - Spruce ID, very involved with DIF on hospitality and travel, authoring schemas


Notes:
Chris Bramwell:
Introductions
Description of November summit.

George McEwan:
Delegation very interesting in healthcare
- head of household

Christopher Bramwell
Definition of Identity
- current in-progress SEDI bill
  https://le.utah.gov/xcode/Title63A/Chapter20/C63A-20_2026050620260506.pdf

Prior bills
- SB260 https://le.utah.gov/~2025/bills/static/SB0260.html
- SB275 https://le.utah.gov/Session/2026/bills/amended/AV_SB0275S01_2026-03-02_10-08-53.pdf


Compromise is going to happen.
When somebody's identity is compromised
First, if someone presents those credentials, duplicity can be detected so that a verifier
has recourse to not accept the duplicitous credential. The individual's identity that is 
compromised must be able to rotate to a next key and that the next key would be accepted
through the market that verifiers would know that the individual who controls the new key
also controlled the previous key.

The end goal is that someone should be able to have a digital identity they control for 
their entire life without having to go back to the government to have them be the basis 
for how they establish their identity.

This is big picture.

Our goal then, is, whatever we select for the core SEDI protocol is that for other
credential formats, how does someone who has a SEDI credential, when they need to present
in another format, such as mDoc at the airport in TSA, how do we facilitate this with
the formats in the broader ecosystem and allow credential formats from the existing
marketplace to be used?