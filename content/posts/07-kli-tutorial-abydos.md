+++
title = "KERI Tutorial Series – Treasure Hunting in Abydos! Issuing and Verifying a Credential (ACDC)"
slug = "kli-tutorial-abydos"
date = "2023-03-09"
updated = "2023-09-27"

[taxonomies]
tags=["tutorial", "keri", "acdc", "said", "decentralized-identity", "ssi", "credentials", "verifiable-credentials"]

[extra]
comment = true
+++

# KLI Tutorial - Abydos Treasure Hunting

{% img_left(path="/images/posts/07-abydos-site.jpg", 
    width=300, height=200,
    alt="Abydos entrance", op="fill") 
    %}
Abydos entrance
{% end %}

Welcome to the treasure hunting adventure! This tutorial demonstrates how to issue and verify Authentic Chained Data Containers (ACDCs) using KERI. Join our intrepid explorers as they navigate the ancient mysteries of Abydos, Egypt, armed with verifiable credentials.

{% image(path="/images/posts/07-abydos-athena-component-graph-1.png", class="diagram",
    op="fill",
    alt="ATHENA Network diagram") 
    %}
ATHENA Network Diagram
{% end %}

**9/27/23 Update**: The Mark I KERIpy agent has been deprecated and removed from the main KERIpy repository. This post is pending an update to use KERIA (Mark II) and IPEX. In the meantime, the KLI workflow still works.

If you haven't yet completed the [KERI Tutorial Series: Sign and Verify with Heartnet](/kli-tutorial-heartnet), I recommend doing that first to familiarize yourself with KERI basics.

## Acknowledgments

{% img_right(path="/images/posts/07-abydos-iiw-logo.png", 
    width=150, height=80,
    alt="IIW Logo", op="fill") 
    %}
IIW
{% end %}

{% img_left(path="/images/posts/07-abydos-keri-logo.png", 
    width=200, height=60,
    alt="KERI Logo", op="fill") 
    %}
KERI
{% end %}

Special thanks to the Internet Identity Workshop (IIW) community and the KERI development team for their contributions to decentralized identity.

# The Story

Our story features four characters in an Egyptian-themed adventure:

{% image(path="/images/posts/07-abydos-richard.png", class="small",
    op="fill",
    alt="Richard") 
    %}
Richard - Explorer
{% end %}

{% image(path="/images/posts/07-abydos-elayne.png", class="small",
    op="fill",
    alt="Elayne") 
    %}
Elayne - Explorer
{% end %}

{% image(path="/images/posts/07-abydos-ramiel.png", class="small",
    op="fill",
    alt="Ramiel") 
    %}
Ramiel - The Wiseman
{% end %}

{% image(path="/images/posts/07-abydos-zaqiel.png", class="small",
    op="fill",
    alt="Zaqiel") 
    %}
Zaqiel - The Gatekeeper
{% end %}

1. **Richard** - An explorer seeking to hunt for treasure at the Osireion
2. **Elayne** - Richard's partner in exploration
3. **Ramiel** - The Wiseman, a credential issuer who grants journey charters
4. **Zaqiel** - The Gatekeeper, who guards access to the Osireion

{% img_right(path="/images/posts/07-abydos-osireion.jpeg", 
    width=300, height=240,
    alt="Osireion", op="fill") 
    %}
The Osireion at Abydos
{% end %}

The Abydos Treasure Hunting Expedition Navigators Assembly (ATHENA) has established a system where explorers must obtain proper credentials before accessing the ancient site. The Wiseman issues Journey Charters to qualified explorers, and the Gatekeeper verifies these credentials before granting access.

# Credential Overview

The ATHENA system issues several types of credentials. Here's an overview of what credentials go to whom:

{% image(path="/images/posts/07-abydos-whattowhom-thj.png", class="diagram",
    op="fill",
    alt="Treasure Hunting Journey credential flow") 
    %}
Treasure Hunting Journey Credential
{% end %}

{% image(path="/images/posts/07-abydos-whattowhom-jmr.png", class="diagram",
    op="fill",
    alt="Journey Mark Rules credential flow") 
    %}
Journey Mark Rules Credential
{% end %}

{% image(path="/images/posts/07-abydos-whattowhom-jm.png", class="diagram",
    op="fill",
    alt="Journey Mark credential flow") 
    %}
Journey Mark Credential
{% end %}

{% image(path="/images/posts/07-abydos-whattowhom-jc.png", class="diagram",
    op="fill",
    alt="Journey Charter credential flow") 
    %}
Journey Charter Credential
{% end %}

{% image(path="/images/posts/07-abydos-whattowhom-present.png", class="diagram",
    op="fill",
    alt="Credential presentation flow") 
    %}
Credential Presentation Flow
{% end %}

## Credential Graph

{% img_left(path="/images/posts/07-abydos-athena-credential-graph.png", 
    width=500, height=485,
    alt="ATHENA credential graph", op="fill") 
    %}
ATHENA Credential Graph
{% end %}

The credential graph shows how the different credentials relate to each other. The Treasure Hunting Journey Charter references the Journey Mark credential through an edge, creating a credential chain.

# Outline

1. Set up your machine for a KERI deployment
2. Start the network services (witnesses and schema server)
3. Initialize keystores and create identifiers
4. Set up credential registries
5. Resolve OOBIs (Out-of-Band Introductions)
6. Issue a credential (Journey Charter)
7. Issue a chained credential with edges and rules
8. Present credentials to the Gatekeeper
9. Revoke a credential

# Understanding ACDCs

Before diving into the implementation, let's understand the key concepts.

## What is an ACDC?

An Authentic Chained Data Container (ACDC) is KERI's approach to verifiable credentials. ACDCs provide:

* **Authenticity** - Cryptographic proof of issuance
* **Chaining** - Credentials can reference other credentials via edges
* **Rich Data** - Support for complex schemas with rules and attestations
* **Revocation** - Issuers can revoke credentials using Transaction Event Logs (TELs)

## ACDC Schema Structure

{% image(path="/images/posts/07-abydos-thj-schema-collapsed.png", class="diagram",
    op="fill",
    alt="ACDC top-level view") 
    %}
ACDC Top-Level Schema View
{% end %}

{% image(path="/images/posts/07-abydos-thj-properties-collapsed.png", class="diagram",
    op="fill",
    alt="ACDC properties section") 
    %}
ACDC Properties Section
{% end %}

ACDC schemas are JSON Schema documents with a special property: they are self-addressing. The `$id` field contains a SAID (Self-Addressing Identifier) that is a hash of the schema itself.

{% image(path="/images/posts/07-abydos-image-3.png", class="diagram",
    op="fill",
    alt="Full un-compacted ACDC") 
    %}
Full Un-compacted ACDC
{% end %}

## Schema Fields

{% image(path="/images/posts/07-abydos-image-12.png", class="diagram",
    op="fill",
    alt="Schema fields") 
    %}
Schema Fields
{% end %}

{% image(path="/images/posts/07-abydos-image-13.png", class="diagram",
    op="fill",
    alt="Schema attributes") 
    %}
Schema Attributes
{% end %}

{% image(path="/images/posts/07-abydos-image-14.png", class="diagram",
    op="fill",
    alt="Edge schema") 
    %}
Edge Schema
{% end %}

{% image(path="/images/posts/07-abydos-image-15.png", class="diagram",
    op="fill",
    alt="Rules schema") 
    %}
Rules Schema
{% end %}

{% image(path="/images/posts/07-abydos-image-16.png", class="diagram",
    op="fill",
    alt="Credential data") 
    %}
Credential Data
{% end %}

{% image(path="/images/posts/07-abydos-image-17.png", class="diagram",
    op="fill",
    alt="Credential edge data") 
    %}
Credential Edge Data
{% end %}

{% image(path="/images/posts/07-abydos-image-18.png", class="diagram",
    op="fill",
    alt="Credential rules data") 
    %}
Credential Rules Data
{% end %}

## Key Components

### Transaction Event Log (TEL)

The TEL tracks the lifecycle of credentials - when they're issued, revoked, or otherwise updated. Each credential registry has its own TEL that is anchored to the issuer's Key Event Log (KEL).

### Edges

Edges are references from one ACDC to another, forming a directed acyclic graph (DAG). This allows for credential chaining, where one credential can reference or depend on another.

### Rules

Rules sections in ACDCs can contain Ricardian contracts - legal prose that describes the terms and conditions of the credential.

# Tutorial

Let's get to it!

## Prerequisites

Before starting this tutorial, you should have:

* Completed the [Heartnet tutorial](/kli-tutorial-heartnet) or have equivalent KERI knowledge
* Docker installed (recommended) or a local Python environment with KERI installed
* Git for cloning the repository

## Code Repository

Start by cloning the Git repository. Once you have cloned this repository you can find all commands from this post in the workflow script.

```bash
git clone https://github.com/WebOfTrust/athena.git
cd athena
```

## General Notes

* The `$` dollar sign character indicates a user shell. You do not need to type the `$` character when it appears at the beginning of a shell command line.
* I make frequent use of the shell continuation character `\` to provide readability to commands.
* Output of commands is typically included in code highlighting blocks directly after the command with a header of `Output:`.

# Step 1: Machine Setup

You need to set up your machine to have KERI installed. You can either do this with Docker or with a local installation of Python.

## Docker Setup

This is the recommended approach.

```bash
docker pull weboftrust/keri:1.1.0
```

Verify the installation:

```bash
docker run --rm -it weboftrust/keri:1.1.0 version
```

## Manual Setup

If you prefer a local installation, follow the setup instructions from the [Heartnet tutorial](/kli-tutorial-heartnet).

For this tutorial, you'll also need:

```bash
pip install kaslcred
pip install vLEI
```

# Step 2: Start Network Services

{% image(path="/images/posts/07-abydos-athena-component-graph.png", class="diagram",
    op="fill",
    alt="ATHENA network component diagram") 
    %}
ATHENA Network Component Diagram
{% end %}

The ATHENA system requires several services to be running:

1. Witness network
2. Schema caching server (vLEI-server)

## Start Schema Caching Server

{% img_left(path="/images/posts/07-abydos-athena-caching-server.png", 
    width=400, height=400,
    alt="vLEI-server", op="fill") 
    %}
vLEI Server (Schema Caching Server)
{% end %}

The vLEI-server acts as a schema caching server for ACDC schemas:

```bash
vLEI-server \
    -s "${ATHENA_DIR}/saidified_schemas" \
    -c "${ATHENA_DIR}/cache/acdc" \
    -o "${ATHENA_DIR}/cache/oobis" &
```

This server:
- Serves schemas from the saidified_schemas directory
- Caches ACDCs in the cache/acdc directory
- Caches OOBIs in the cache/oobis directory

## Start Witness Network

{% image(path="/images/posts/07-abydos-athena-witconfig.png", class="diagram",
    op="fill",
    alt="Witness network with config files") 
    %}
Witness Network Configuration
{% end %}

```bash
kli witness demo &
```

Output:
```
Witness wan : BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha
Witness wil : BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM
Witness wes : BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX
Witness wit : BM35JN8XeJSEfpxopjn5jr7tAHCE5749f0OobhMLCorE
Witness wub : BIj15u5V11bkbtAxMA7gcNJZcax-7TgaBMLsQnMHpYHP
Witness wyz : BF2rZTW79z4IXocYRQnjjsOuvFUQv-ptCf8Yltd7PfsM
```

{% image(path="/images/posts/07-abydos-image-4.png", class="diagram",
    op="fill",
    alt="Witness output") 
    %}
Witness Output
{% end %}

Store the witness AIDs for later use:

```bash
export WAN_WITNESS_AID="BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha"
export WIL_WITNESS_AID="BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM"
export WES_WITNESS_AID="BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
```

{% image(path="/images/posts/07-abydos-athena-witkeystores.png", class="diagram",
    op="fill",
    alt="Witness keystores") 
    %}
Witness Keystores
{% end %}

# Step 3: Creating ACDC Schemas

Before issuing credentials, you need to create and "saidify" the schemas.

## Schema Design

Our ATHENA system uses three credential schemas:

1. **Abydos Treasure Hunting Journey Mark** - A simple mark credential
2. **Treasure Hunting Journey Charter** - The main credential for explorers
3. **Treasure Hunting Journey Charter Credential Rules** - The rules schema

## Saidifying Schemas

Use KASLCred to process schemas and populate SAID fields:

```bash
kaslcred saidify \
    --schema-dir "${ATHENA_DIR}/schemas" \
    --output-dir "${ATHENA_DIR}/saidified_schemas"
```

This tool:
1. Calculates SAIDs for each schema
2. Populates `$id` fields with calculated SAIDs
3. Orders schemas by dependency for proper resolution

# Step 4: Initialize Keystores and Create Identifiers

{% image(path="/images/posts/07-abydos-athena-ctlr-net.png", class="diagram",
    op="fill",
    alt="Controller network") 
    %}
Controller Network
{% end %}

We need to create keystores and identifiers for each participant.

{% image(path="/images/posts/07-abydos-image-5.png", class="diagram",
    op="fill",
    alt="Keystore initialization output") 
    %}
Keystore Initialization
{% end %}

## Agent Architecture

{% img_left(path="/images/posts/07-abydos-athena-agent.png", 
    width=300, height=300,
    alt="KERI Agent", op="fill") 
    %}
KERI Agent
{% end %}

Each participant runs a KERI agent that manages their keys, keystores, and credentials.

{% image(path="/images/posts/07-abydos-athena-agents.png", class="diagram",
    op="fill",
    alt="All agents") 
    %}
All ATHENA Agents
{% end %}

## Explorer (Richard)

### Initialize Keystore

{% image(path="/images/posts/07-abydos-athena-ctlr-keystores.png", class="diagram",
    op="fill",
    alt="Controller keystores") 
    %}
Controller Keystores
{% end %}

```bash
EXPLORER_SALT="0AAiU3Ih3WYmTuWWymZTYFbP"
EXPLORER_KEYSTORE="richard"
EXPLORER_ALIAS="richard"

kli init \
    --name ${EXPLORER_KEYSTORE} \
    --salt ${EXPLORER_SALT} \
    --nopasscode \
    --config-file "${ATHENA_DIR}/config/explorer-keystore-config.json"
```

Output:
```
KERI Keystore created at: /usr/local/var/keri/ks/richard
KERI Database created at: /usr/local/var/keri/db/richard
KERI Credential Store created at: /usr/local/var/keri/reg/richard

Loading 3 OOBIs...
http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha/controller succeeded
http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM/controller succeeded
http://127.0.0.1:5644/oobi/BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX/controller succeeded
```

### Create Identifier

{% image(path="/images/posts/07-abydos-athena-ctlr-incept.png", class="diagram",
    op="fill",
    alt="Controller inception") 
    %}
Controller Inception
{% end %}

The inception configuration file:

```json
{
  "transferable": true,
  "wits": [
    "BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "toad": 2,
  "icount": 1,
  "ncount": 1,
  "isith": "1",
  "nsith": "1"
}
```

Create the inception event:

```bash
kli incept \
    --name ${EXPLORER_KEYSTORE} \
    --alias ${EXPLORER_ALIAS} \
    --file "${ATHENA_DIR}/config/explorer-inception-config.json"
```

Output:
```
Waiting for witness receipts...
Prefix  EGvBQV9_luahWp9GrgsxLFSdHxAnH3qlRJgmL8TnB5o0
    Public key 1:  DJiPjlqjbvC4LxzlLp5bu6vVfnb0vnK6bUG7vGtmfMU2
```

Store the prefix:

```bash
export RICHARD_PREFIX="EGvBQV9_luahWp9GrgsxLFSdHxAnH3qlRJgmL8TnB5o0"
```

## Wiseman (Ramiel)

### Initialize Keystore

```bash
WISEMAN_SALT="0ABfYE2dBj96dT9MNMFIT4Fw"
WISEMAN_KEYSTORE="ramiel"
WISEMAN_ALIAS="ramiel"

kli init \
    --name ${WISEMAN_KEYSTORE} \
    --salt ${WISEMAN_SALT} \
    --nopasscode \
    --config-file "${ATHENA_DIR}/config/wiseman-keystore-config.json"
```

### Create Identifier

```bash
kli incept \
    --name ${WISEMAN_KEYSTORE} \
    --alias ${WISEMAN_ALIAS} \
    --file "${ATHENA_DIR}/config/wiseman-inception-config.json"
```

Output:
```
Waiting for witness receipts...
Prefix  EKQsMBJHhD8j2F76BFSphBqU1SdqKb0eDEtTCdz0Z4Es
    Public key 1:  DPmhqrPkOd_8y4NJnm8qLN5zVe3kbMXf6dJ_R4h1lZnA
```

Store the prefix:

```bash
export WISEMAN_PREFIX="EKQsMBJHhD8j2F76BFSphBqU1SdqKb0eDEtTCdz0Z4Es"
```

## Gatekeeper (Zaqiel) - Custom Controller

The Gatekeeper uses Sally, a custom controller framework built on KERIpy. Sally allows you to add custom business logic for credential verification.

### Start Gatekeeper Service

```bash
sally server start \
    --name zaqiel \
    --alias zaqiel \
    --web-hook http://localhost:9923 \
    --auth "${WISEMAN_PREFIX}" &
```

The Gatekeeper:
- Runs as a service listening for credential presentations
- Validates incoming credentials against expected schemas
- Calls a webhook when credentials are presented
- Can enforce business rules beyond simple cryptographic verification

# Step 5: Resolve OOBIs

{% image(path="/images/posts/07-abydos-athena-oobis-all.png", class="diagram",
    op="fill",
    alt="All OOBIs") 
    %}
All OOBI Resolutions
{% end %}

Participants need to discover each other before they can exchange credentials.

## Schema OOBI Resolution

First, resolve the schema OOBIs so all participants know about the credential schemas:

```bash
SCHEMA_SERVER_URL="http://127.0.0.1:7723"
JOURNEY_CHARTER_SCHEMA_SAID="EBfdlu8R27Fbx-ehrqwImnK-8Cm79sqbAQ4MmvEAYqao"

kli oobi resolve \
    --name ${EXPLORER_KEYSTORE} \
    --oobi-alias journey-charter-schema \
    --oobi "${SCHEMA_SERVER_URL}/oobi/${JOURNEY_CHARTER_SCHEMA_SAID}"
```

## Participant OOBI Resolution

{% image(path="/images/posts/07-abydos-athena-oobis-richard-ramiel.png", class="diagram",
    op="fill",
    alt="Richard and Ramiel OOBI resolution") 
    %}
Richard and Ramiel OOBI Resolution
{% end %}

### Wiseman Resolves Explorer OOBI

```bash
WAN_WITNESS_URL="http://127.0.0.1:5642"

kli oobi resolve \
    --name ${WISEMAN_KEYSTORE} \
    --oobi-alias ${EXPLORER_ALIAS} \
    --oobi "${WAN_WITNESS_URL}/oobi/${RICHARD_PREFIX}/witness/${WAN_WITNESS_AID}"
```

Output:
```
http://127.0.0.1:5642/oobi/EGvBQV9_luahWp9GrgsxLFSdHxAnH3qlRJgmL8TnB5o0/witness/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha resolved
```

### Explorer Resolves Wiseman OOBI

```bash
kli oobi resolve \
    --name ${EXPLORER_KEYSTORE} \
    --oobi-alias ${WISEMAN_ALIAS} \
    --oobi "${WAN_WITNESS_URL}/oobi/${WISEMAN_PREFIX}/witness/${WAN_WITNESS_AID}"
```

{% image(path="/images/posts/07-abydos-athena-oobis-witness.png", class="diagram",
    op="fill",
    alt="Witness OOBIs") 
    %}
Witness OOBIs
{% end %}

### Explorer Resolves Gatekeeper OOBI

```bash
GATEKEEPER_PREFIX="EHpD0-CDWOdu5RJ8jHBSUkOqBZ3cXeDVHWNb_Ul89VI7"
GATEKEEPER_ALIAS="zaqiel"

kli oobi resolve \
    --name ${EXPLORER_KEYSTORE} \
    --oobi-alias ${GATEKEEPER_ALIAS} \
    --oobi "${WAN_WITNESS_URL}/oobi/${GATEKEEPER_PREFIX}/witness/${WAN_WITNESS_AID}"
```

# Step 6: Set Up Credential Registries

{% image(path="/images/posts/07-abydos-athena-registry-ramiel.png", class="diagram",
    op="fill",
    alt="Ramiel's registry") 
    %}
Ramiel's Credential Registry
{% end %}

Before issuing credentials, each issuer needs a credential registry.

## Create Wiseman Registry

{% img_right(path="/images/posts/07-abydos-blue-credential.jpg", 
    width=200, height=150,
    alt="Credential", op="fill") 
    %}
Credential
{% end %}

```bash
WISEMAN_REGISTRY="wiseman-registry"

kli vc registry incept \
    --name ${WISEMAN_KEYSTORE} \
    --alias ${WISEMAN_ALIAS} \
    --registry-name ${WISEMAN_REGISTRY}
```

Output:
```
Waiting for TEL event witness receipts
Registry EHZPsYZw7dTMxJH5oTi9jCzm9QbEP4a0_9R6E6p8Qxq8 created for ramiel
```

{% image(path="/images/posts/07-abydos-athena-registries.png", class="diagram",
    op="fill",
    alt="All registries") 
    %}
All Credential Registries
{% end %}

# Step 7: Issue a Credential

{% img_left(path="/images/posts/07-abydos-blue-credential.jpg", 
    width=200, height=150,
    alt="Credential", op="fill") 
    %}
Issuing a Credential
{% end %}

Now the Wiseman can issue a Journey Charter credential to Richard.

## Prepare Credential Data

Create a JSON file with the credential attributes:

```json
{
  "explorerName": "Richard",
  "expeditionId": "ATHENA-2023-001",
  "LEI": "254900OPPU84GM83MG36"
}
```

Save this to `credential_data/journey-charter-richard.json`.

## Issue the Credential

{% image(path="/images/posts/07-abydos-credentials-all.png", class="diagram",
    op="fill",
    alt="All credentials") 
    %}
All Credentials Overview
{% end %}

```bash
kli vc issue \
    --name ${WISEMAN_KEYSTORE} \
    --alias ${WISEMAN_ALIAS} \
    --registry-name ${WISEMAN_REGISTRY} \
    --schema "${JOURNEY_CHARTER_SCHEMA_SAID}" \
    --recipient "${RICHARD_PREFIX}" \
    --data @"${ATHENA_DIR}/credential_data/journey-charter-richard.json"
```

Output:
```
Waiting for TEL event witness receipts
Credential EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w issued to EGvBQV9_luahWp9GrgsxLFSdHxAnH3qlRJgmL8TnB5o0
```

Store the credential SAID:

```bash
export RICHARD_JOURNEY_CHARTER_CRED_SAID="EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w"
```

## Explorer Receives the Credential

{% image(path="/images/posts/07-abydos-credentials-thj.png", class="diagram",
    op="fill",
    alt="Treasure Hunting Journey credentials") 
    %}
Treasure Hunting Journey Credentials
{% end %}

Richard needs to poll for and accept the credential:

```bash
kli vc list \
    --name ${EXPLORER_KEYSTORE} \
    --alias ${EXPLORER_ALIAS} \
    --poll
```

Output:
```
Credential EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w:
    Issuer: EKQsMBJHhD8j2F76BFSphBqU1SdqKb0eDEtTCdz0Z4Es (ramiel)
    Schema: EBfdlu8R27Fbx-ehrqwImnK-8Cm79sqbAQ4MmvEAYqao
    Status: Issued
```

# Step 8: Issue a Chained Credential with Edges and Rules

{% image(path="/images/posts/07-abydos-athena-creds-thj-to-jmr.png", class="diagram",
    op="fill",
    alt="Credentials THJ to JMR flow") 
    %}
Credentials Flow: THJ to JMR
{% end %}

ACDCs support chaining through edges. Let's issue a more complex credential that references another credential.

## Understanding Edges

Edges create links between ACDCs, forming a directed acyclic graph (DAG). This enables:

* **Provenance chains** - Track the origin of authorizations
* **Conditional credentials** - Credentials that depend on other credentials
* **Delegation** - Authority granted based on existing credentials

{% image(path="/images/posts/07-abydos-credentials-jmr.png", class="diagram",
    op="fill",
    alt="JMR credentials") 
    %}
Journey Mark Rules Credentials
{% end %}

## Issue Chained Credential

First, prepare the edge data referencing Richard's journey charter:

```json
{
  "charter": {
    "n": "EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w",
    "s": "EBfdlu8R27Fbx-ehrqwImnK-8Cm79sqbAQ4MmvEAYqao"
  }
}
```

Then issue the chained credential:

```bash
kli vc issue \
    --name ${WISEMAN_KEYSTORE} \
    --alias ${WISEMAN_ALIAS} \
    --registry-name ${WISEMAN_REGISTRY} \
    --schema "${EXPEDITION_AUTHORIZATION_SCHEMA_SAID}" \
    --recipient "${RICHARD_PREFIX}" \
    --data @"${ATHENA_DIR}/credential_data/expedition-auth-richard.json" \
    --edges @"${ATHENA_DIR}/credential_data/expedition-auth-edges.json" \
    --rules @"${ATHENA_DIR}/credential_data/expedition-auth-rules.json"
```

{% image(path="/images/posts/07-abydos-image-6.png", class="diagram",
    op="fill",
    alt="Credential issuance output") 
    %}
Credential Issuance Output
{% end %}

## Issue Journey Mark Credential

{% image(path="/images/posts/07-abydos-credentials-jm.png", class="diagram",
    op="fill",
    alt="Journey Mark credentials") 
    %}
Journey Mark Credentials
{% end %}

{% image(path="/images/posts/07-abydos-credentials-all-1.png", class="diagram",
    op="fill",
    alt="All credentials final") 
    %}
All Credentials - Final State
{% end %}

# Step 9: Present Credentials to the Gatekeeper

Richard must present his credential to the Gatekeeper to gain access to the Osireion.

## Present Credential

{% image(path="/images/posts/07-abydos-image-7.png", class="diagram",
    op="fill",
    alt="Credential presentation") 
    %}
Credential Presentation
{% end %}

```bash
kli vc present \
    --name ${EXPLORER_KEYSTORE} \
    --alias ${EXPLORER_ALIAS} \
    --said "${RICHARD_JOURNEY_CHARTER_CRED_SAID}" \
    --recipient ${GATEKEEPER_ALIAS} \
    --include
```

Output:
```
Credential EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w presented to EHpD0-CDWOdu5RJ8jHBSUkOqBZ3cXeDVHWNb_Ul89VI7
```

The `--include` flag tells the command to include the full credential in the presentation, not just a reference.

## Gatekeeper Verification

{% image(path="/images/posts/07-abydos-image-9.png", class="diagram",
    op="fill",
    alt="Gatekeeper verification") 
    %}
Gatekeeper Verification
{% end %}

The Sally-based Gatekeeper:

1. Receives the presentation
2. Verifies the credential signature chain
3. Checks the TEL for revocation status
4. Validates against expected schema
5. Executes custom business logic
6. Calls the configured webhook with results

{% image(path="/images/posts/07-abydos-image-8.png", class="diagram",
    op="fill",
    alt="Verification output") 
    %}
Verification Output
{% end %}

### Custom Verification Logic

Sally allows custom handlers for credential types. Here's an example handler:

```python
def handleJourneyCharterPresentation(self, creder, attachments):
    """Handle presentation of a Journey Charter credential"""
    
    # Extract credential attributes
    attrs = creder.attribs
    explorer_name = attrs.get('explorerName')
    expedition_id = attrs.get('expeditionId')
    
    # Custom business logic
    if not self.validateExpeditionId(expedition_id):
        return {"status": "rejected", "reason": "Invalid expedition ID"}
    
    # Check if explorer is on approved list
    if not self.isApprovedExplorer(explorer_name):
        return {"status": "rejected", "reason": "Explorer not approved"}
    
    # Grant access
    return {
        "status": "approved",
        "explorer": explorer_name,
        "expedition": expedition_id,
        "accessLevel": "osireion"
    }
```

{% image(path="/images/posts/07-abydos-image-10.png", class="diagram",
    op="fill",
    alt="Webhook response") 
    %}
Webhook Response
{% end %}

{% image(path="/images/posts/07-abydos-image-11.png", class="diagram",
    op="fill",
    alt="Full verification flow") 
    %}
Full Verification Flow
{% end %}

# Step 10: Revoke a Credential

{% img_left(path="/images/posts/07-abydos-athena-credential-graph.png", 
    width=400, height=400,
    alt="Credential graph", op="fill") 
    %}
Credential Graph - Revocation Context
{% end %}

The Wiseman can revoke a credential if needed.

## Revoke Credential

```bash
kli vc revoke \
    --name ${WISEMAN_KEYSTORE} \
    --alias ${WISEMAN_ALIAS} \
    --registry-name ${WISEMAN_REGISTRY} \
    --said "${RICHARD_JOURNEY_CHARTER_CRED_SAID}" \
    --send ${GATEKEEPER_ALIAS}
```

Output:
```
Waiting for TEL event witness receipts
Credential EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w revoked
Sending revocation notice to zaqiel
```

The `--send` flag sends a revocation notice to the specified recipient(s).

## Verify Revocation

After revocation, the Gatekeeper's verification will fail:

```bash
kli vc list \
    --name ${EXPLORER_KEYSTORE} \
    --alias ${EXPLORER_ALIAS} \
    --poll
```

Output:
```
Credential EJFfdHpZHCQMxk5kZOoX2hLvV3vT8snYhLPOx5W3TW0w:
    Issuer: EKQsMBJHhD8j2F76BFSphBqU1SdqKb0eDEtTCdz0Z4Es (ramiel)
    Schema: EBfdlu8R27Fbx-ehrqwImnK-8Cm79sqbAQ4MmvEAYqao
    Status: Revoked
```

# Understanding the Protocols

## PTEL Protocol

The Public Transaction Event Log (PTEL) protocol manages credential lifecycle:

* **Issuance (iss)** - Records credential creation
* **Revocation (rev)** - Records credential revocation
* **Backers** - Witnesses that observe TEL events

## IPEX Protocol

The Issuance and Presentation Exchange (IPEX) protocol defines the workflow for:

* **Apply** - Request a credential
* **Offer** - Offer to issue a credential
* **Agree** - Accept an offer
* **Grant** - Issue the credential
* **Admit** - Acknowledge receipt
* **Present** - Present a credential
* **Spurn** - Reject a presentation

# Wrap Up

{% image(path="/images/posts/07-abydos-treasure-hunting.jpg", class="diagram",
    op="fill",
    alt="Treasure hunting complete") 
    %}
Treasure Hunting Complete!
{% end %}

You have successfully completed the ATHENA treasure hunting expedition! You've learned:

## Key Concepts Covered

1. **ACDC Structure** - Authentic Chained Data Containers and their components
2. **SAIDs** - Self-Addressing Identifiers for schemas and credentials
3. **Credential Registries** - TELs for tracking credential lifecycle
4. **Schema Design** - Creating and saidifying JSON Schema credentials
5. **OOBI Resolution** - Discovering schemas and participants
6. **Credential Issuance** - Issuing simple and chained credentials
7. **Edges and Rules** - Creating credential chains with legal terms
8. **Credential Presentation** - Presenting credentials to verifiers
9. **Custom Controllers** - Using Sally for business logic
10. **Credential Revocation** - Revoking and notifying about revoked credentials

## Next Steps

* Explore the [ACDC specification](https://github.com/trustoverip/acdc)
* Learn about [KERIA](https://github.com/WebOfTrust/keria) - the Mark II agent
* Build your own credential schema for your use case
* Implement custom verification logic with Sally

See you in the next tutorial!

## References

- [KERI](https://keri.one/) Homepage
- [ACDC Specification](https://github.com/trustoverip/acdc) on GitHub
- [KASLCred](https://github.com/WebOfTrust/kaslcred) - Schema processing tool
- [Sally](https://github.com/WebOfTrust/sally) - Custom controller framework
- [vLEI](https://github.com/GLEIF-IT/vLEI) - Verifiable LEI implementation
- KERI Terms [Wiki](https://github.com/trustoverip/acdc/wiki)
