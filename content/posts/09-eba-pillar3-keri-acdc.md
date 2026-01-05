+++
title = "European Banking Association with Pillar 3 data hub adopts KERI & ACDC"
slug = "eba-pillar3-keri-acdc"
date = "2023-12-15"

[taxonomies]
tags=["keri", "acdc", "eba", "european-banking-authority", "finance", "fintech", "pillar-3", "vlei"]

[extra]
comment = true
+++

# European Banking Association with Pillar 3 data hub adopts KERI & ACDC

There is some *major* positive good news for the KERI & ACDC ecosystem. The European Banking Authority announced on LinkedIn the EBA Pillar 3 data hub which is built on KERI & ACDC. The Pillar 3 data hub uses the vLEI system developed by GLEIF, the Global Legal Entity Identifier Foundation.

{% image(path="/images/posts/09-eba-vlei-architecture.png", class="diagram",
    op="fit_width",
    alt="vLEI Architecture") 
    %}
vLEI Architecture Diagram
{% end %}

## What this means

This means that around 6000 financial institutions in the EU will now be required to use the EBA Pillar 3 data hub, and by extension KERI & ACDC, for their financial reporting. You can read more in their [website announcement](https://www.eba.europa.eu/publications-and-media/press-releases/eba-publishes-discussion-paper-pillar-3-data-hub) and the linked [report PDF](https://www.eba.europa.eu/sites/default/files/2023-12/d5b13b4d-a9dc-4680-8b7c-0a3a4c694fac/Discussion%20paper%20on%20Pillar3%20data%20hub.pdf).

The important parts include pages 50-52, specifically the Gartner analysis in section 6.3.128:

- EBA had Gartner analyze KERI & ACDC
- found "that there are no comparably efficient alternative solutions globally"

And section 6.4.130 Conclusion:

- "vLEI effectively meets…Pillar 3 reporting requirements…perceived as a low-risk project overall…significant opportunity to enhance integrity of reporting processes"

A number of advantages and risks were identified with the paper saying:

> The automation of identity verification and related processes through the vLEI could also offer numerous potential advantages for both Financial Institutions and other Regulators in the EU financial market

## Potential Advantages

### For Financial Institutions

- **Non-repudiable identification**
  - This means once you say something with KERI you can't un-say it, as in you can't change your story without getting caught
- **Operational efficiency**: unified digital format
- **Enhanced products and services**: trusted information enables risk management, customer service, and enhanced online experience

### For Supervisors/Regulators

- **Enhanced Trust**: simplify both validation of regulatory reports and authorized sign off
- **Comprehensive Entity Overview**: transparent, aggregated view of legal entities and hierarchies
- **Standardization of Data Processes**: smarter, more cost-effective, reliable data workflows

## Risks

### Development of ecosystem

- Due to ecosystem novelty there is only one QVI right now (Provenant)
  - Support is needed from GLEIF to grow the population of QVIs, incentive structure, and ecosystem orchestration
- Ensure adequate support to institutions
  - End-user applications bank wallet
  - Wallet for reporting institutions
  - Compatibility with eIDAS 2.0 framework
  - User-friendly applications for digital signatures, key management, and logging services
- Market's recognition of benefits offered by vLEI
  - Pillar 3 reporting requirement could trigger a "snowball effect" to catalyze market recognition of vLEI benefits.

This is an exciting development in the KERI space that this blog will continue to track.
