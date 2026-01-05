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

# KERI Tutorial Series: Treasure Hunting in Abydos! Issuing and Verifying a Credential (ACDC)



{% img_left(path="/images/posts/07-abydos-site.jpg", 
    width=300, height=200,
    alt="Abydos entrance", op="fit_width") 
    %}
Abydos entrance
{% end %}

Welcome to the latest installment in the KERI Tutorial Series! In a [prior article](https://www.kentbull.com/2023/01/27/keri-tutorial-series-kli-sign-and-verify-with-heartnet/) we learned how to use KERI to sign and verify simple messages. This tutorial goes to the full depth of issuing and verifying credentials (ACDCs).

We outfit our team of explorers bound on a treasure hunting journey with high tech security tags so the Gatekeeper will recognize them and permit them entrance into the most rewarding, yet dangerous, caverns of the secret underground chambers beneath the Osireion at Abydos.

#### Update (warning): KERIpy agent now removed. Post pending update.

09/27/2023: As of KERIpy [pull request 577](https://github.com/WebOfTrust/keripy/pull/577/files) the old, deprecated KERIpy agent (Mark I agent) used in this blog post is now removed. This post will be updated to use the new agent which is KERIA as well as the new IPEX protocol for credential handling (Issuance and Presentation Exchange).

## Egyptian Story Time

An ancient city in Egypt from the time of the Pharaohs Abydos has long been known as a burial and memorial site for the ancient rulers. Yet, until recently, what had been kept a secret is the true purpose of Abydos and the Osireion, an entrance to an underground world full of bounty and treasure where the Egyptian rulers stored their most precious things.

Even with this revelation few could master the secret of how to travel between the Osireion and this underground world. Only two ever reliably figured it out, first the wise Ramiel and then his strong student Zaqiel. After they retrieved all the riches and treasure they wanted they realized they had an opportunity on their hands as guides for future explorers. Thus ATHENA was born.

### Opportunity

To establish a new standard in treasure hunting trust, and to capitalize on their opportunity, Ramiel and Zaqiel formed the **Abydos Treasure Hunting Expedition Navigators Assembly** (ATHENA) and began looking for a technology partner to build their *Journey Mark trust system*. Ramiel took on the role of the Wise Man to evaluate, grant marks to, and guide treasure hunters to Abydos while Zaqiel, the stronger of the two, took on the role of the Osireion Gatekeeper to ensure only those accepted by Ramiel would be given the opportunity to learn to travel to the underground world.

## What ATHENA seeks to create

According to ATHENA, explorers desiring entrance through the Osireion must present a verifiable, secure proof signed by the Wise Man in order to be accepted by the Gatekeeper. Potential explorers must signify their intent to embark on a journey by presenting the Wise Man with another secure proof.

The diagram below shows the end state of their desired network.

{% image(path="/images/posts/07-abydos-athena-component-graph-1.png", class="diagram",
    op="fit_width",
    alt="ATHENA Network diagram") 
    %}
ATHENA Network diagram
{% end %}

On a practical note, get some refreshments and settle in as this post is large and contains many parts. It may take a few sessions to fully absorb the material depending on your existing familiarity with KERI and ACDC and the related Github repositories.

### You meet the two

So how did you ever come to meet up with Ramiel and Zaqiel at ATHENA? It turns out the story is more believable than you might think.

{% img_right(path="/images/posts/07-abydos-iiw-logo.png", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}

While on a casual stroll at the Internet Identity Workshop (IIW) you had the good fortune of bumping into Ramiel and Zaqiel and they asked you how you would build the ATHENA security system. Since you had just left Dr. Samuel Smith’s session on composable event streaming representation (CESR) and had earlier been to Drummond Reed’s session KERI for Muggles where you learned about key event receipt infrastructure (KERI) and authentic chained data containers (ACDC), you realized they were the best decentralized key management infrastructure (DKMI) available and figured this would be the right tool to get the job done for ATHENA.

{% img_left(path="/images/posts/07-abydos-keri-logo.png", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}

After hearing the details from Ramiel you became excited about the opportunity. You enthusiastically shook hands with them and began your work. Then, feeling a little in over your head and hoping for a concise and useful guide on KERI and ACDC to get you started on your development path, you hear Kaliya Identity Woman Young casually mention a link in the session notes she hounded the session presenter to write down. Grateful for her efforts you began to feel a sense of relief as you turned to the IIW QiQo site and found this blog post. A rush of hope and encouragement came and you dug in realizing you found the KERI manual you needed to get a simple prototype up and running.

Your job is to make this all happen.

## What you can expect to learn
- All of the components that are required for a basic KERI network including configuration
- ACDC Credential Schema Creation, Issuance, and Revocation
- KERI Command Line Interface (kli) and the (now deprecated) KERI Mark I Agent REST API
- To learn about the KERIpy code. There are multiple references to important parts of the KERIpy codebase strewn throughout this tutorial.
- A fair bit about BASH scripting if you read through the workflow.sh script.


## Characters

Our Explorer team includes the following cast of characters:

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 1.5rem; max-width: 400px; margin: 1.5rem 0;">
  <div style="text-align: center;">
    <img src="/images/posts/07-abydos-richard.png" alt="Richard the Explorer" style="width: 100px; height: 100px; object-fit: contain;">
    <p style="margin-top: 0.5rem; font-weight: bold;">Richard the Explorer</p>
  </div>
  <div style="text-align: center;">
    <img src="/images/posts/07-abydos-elayne.png" alt="Elayne the Librarian" style="width: 100px; height: 100px; object-fit: contain;">
    <p style="margin-top: 0.5rem; font-weight: bold;">Elayne the Librarian</p>
  </div>
</div>

The ATHENA team includes:

<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 1.5rem; max-width: 400px; margin: 1.5rem 0;">
  <div style="text-align: center;">
    <img src="/images/posts/07-abydos-ramiel.png" alt="Ramiel the Wise Man" style="width: 100px; height: 100px; object-fit: contain;">
    <p style="margin-top: 0.5rem; font-weight: bold;">Ramiel the Wise Man</p>
  </div>
  <div style="text-align: center;">
    <img src="/images/posts/07-abydos-zaqiel.png" alt="Zaqiel the Gatekeeper" style="width: 100px; height: 100px; object-fit: contain;">
    <p style="margin-top: 0.5rem; font-weight: bold;">Zaqiel the Gatekeeper</p>
  </div>
</div>

Now that you've met the team let's get going. We have a lot to do!

## Outline

As you can see we start with a visual of where we are going, cover concepts briefly, perform environment setup, and then a quick review of major terminology, and it is off to the races with the admittance workflow. From schema writing to network setup and finally to issuance, presentation, and revocation this tutorial takes you through the basics for using KERI and ACDC.
1. Goal Network – End State ATHENA Network
2. Foundational Vocabulary and Theory Keys, KELs, ACDCs, TELs
3. Schemas, SAIDs, OOBIs
4. Controllers, Witnesses,
5. Runtime Environment Setup – Dependency Installation
6. Abydos Admittance Workflow
7. Credential Map
8. ACDC Schema Writing
9. Schema Linking and Verification
10. Network Configuration
11. Witness Network
12. KERI Controller Setup Write controller configuration file
13. KERI Agents Agent flow: start agents
14. KERI Keystores and Inception
15. Out of Band Introductions Between Controllers – OOBIs
16. Credential Registries
17. Credential Issuance
18. Advanced Credential Issuance – with Edges and Rules
19. Credential Presentation Abydos Gatekeeper: Custom controller for credential handlers
20. Webhook Setup
21. Credential Revocation
22. Missing Parts
23. Review


To cover all of these items in this already very large post we must get going so we will leave out a significant list of concepts that will be addressed with future articles.

### Concepts not covered
- Key Rotation, Reserve Rotation, Custodial rotation
- KERI Data structures in depth
- KERI Message Seals
- KERI Event Messages (Key Event, Receipt, Query, Reply, KSN, TSN, TEL, etc.)
- Edge signing with mobile applications
- Multi-signature groups
- Composable event streaming representation (CESR)
- `did:keri` and `did:webs`
- Watcher networks and other roles including judges and jurors
- KERI Mark II Agent
- KERI vocabulary in depth – we only go to a shallow depth in this post for brevity reasons
- Public Transaction Event Logs (PTEL)
- Duplicity detection and recovery from duplicity


There is a lot to KERI though this article gets you set up with the foundation. Once you are up and running with the contents of this blog post you will have a foundation upon which to explore the other topics listed above.

For now, let’s go to Abydos!

## Going to the Osireion at Abydos

{% img_right(path="/images/posts/07-abydos-osireion.jpeg", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}

### Our Task
1. Create the network foundation.
2. Create digital identities for each involved party.
3. Give a verifiable journey description to each interested explorer.
4. Request a Journey Mark for each explorer from the Wise Man.
5. As the Wise Man issue the Journey Mark to each member of the explorer party.
6. Present the Journey Marks to the Gatekeeper and grant access to the Osireion at Abydos.


### Environment Setup

Check your environment-specific platform installation instructions for additional clarity though use the “All Platforms” list for a list of dependencies that ultimately must be installed on your machine and available from the command line.

#### All Platforms
1. Python 3.10+ required
2. Download the [Abydos Tutorial repository](https://github.com/TetraVeda/abydos-tutorial.git) with all of the example code:`$ git clone https://github.com/TetraVeda/abydos-tutorial`
3. Install Rust (for the blake3 dependency KERIpy depends on and builds)`$` `curl https://sh.rustup.rs -sSf | bash -s -- -y && source "$HOME/.cargo/env"`
4. Install Maturin (for package management one of KERIpy’s dependencies uses)`$` `pip install maturin`
5. Install KERIpy at commit hash “6b2c9868283f57d73a5011607d62135a4d3039ca”, or March 10, 2023You must install from source by doing a `$` `git clone https://github.com/WebOfTrust/keripy` And then `$` `cd keripy`change branches with$ git checkout -b mar-10-2023 6b2c9868283f57d73a5011607d62135a4d3039caand then install KERI with`$` `python -m pip install -e./`This will give you the latest version which should be 1.0.0.
6. Install KASLCred: [https://pypi.org/project/kaslcred/](https://pypi.org/project/kaslcred/)`$` `pip install kaslcred`==0.1.0
7. Install at least version 0.6.1 of the [kentbull/sally](https://github.com/kentbull/sally) fork of GLEIF-IT/sally. `$` `git clone https://github.com/kentbull/sally.git``$` `cd sally``$` `python -m pip install -e ./`
8. Libsodium [https://libsodium.gitbook.io/doc/installation](https://libsodium.gitbook.io/doc/installation)
9. On my MacBook Pro I ended up having to do the full installation: Download a tarball from the [releases page](https://download.libsodium.org/libsodium/releases/).Change directories to within the decompressed tarball.`$` `./configure``$` `make && make check``$` `sudo make install`
10. vLEI credential caching server (vLEI-server) [https://github.com/WebOfTrust/vLEI.git](https://github.com/WebOfTrust/vLEI.git)
11. Clone the repository and do a Pip install`$ git clone https://github.com/WebOfTrust/vLEI.git``$ cd vLEI``$ python -m pip install -e ./`


#### OS X
1. With [Homebrew](https://brew.sh/): Install Python 3.10+, Python 3 PIP, Python 3 Venv, Libsodium-dev, maturin, and the Rust toolchain Installing Python 3.10+ on a Mac is a rather involved process since the operating system default installation can interfere with things. Use the following freeCodeCamp guide to install “pyenv” and then install Python 3 on top of that: [How to Install Python 3 on Mac – Brew Install Update Tutorial](https://www.freecodecamp.org/news/python-version-on-mac-update/)
2. `$ brew install libsodium` # warning, doesn’t work.# See above instructions on libsodium installation from the tarball to get things working on a Mac. You need Libsodium on your path, which `sudo make install` does.
3. `$ pip install maturin` # if you haven’t already
4. Install KERI `$ git clone git clone https://github.com/WebOfTrust/keripy.git`
5. `$ cd keripy`
6. `$ python -m pip install -e ./`
7. Verify your installation `$ kli version`
8. This should output something like “1.0.0”.


#### Windows

Thanks to Jim Martin for providing these Windows instructions.

See the [Manual Setup with Windows](https://www.kentbull.com/2023/01/27/keri-tutorial-series-kli-sign-and-verify-with-heartnet/#windows-manual-setup) section of the KLI with Heartnet tutorial for instructions on setting up KERI on Windows.

#### Jump to the end with the `workflow.sh` script

If you want to quickly start up a network rather than working through the entire blog post piece by piece then you can use the `workflow.sh` script to start the whole network and perform all of the actions including schema linking, configuration file writing, witness network startup, controller configuration and startup, OOBI configuration, credential issuance, and credential revocation.

The workflow script accepts a few different ways of running it:

```bash
# from within the abydos-tutorial directory
#
# Option 1: run with the KLI and no clearing
./workflow.sh
# this will run the entire workflow yet will not clean up the ~/.keri, 
# ~/.sally, or the /usr/local/var/keri directories so you will need
# to manually clear those out in order to run the workflow script again.
#
# Option 2: run with the KLI and the clearing option
./workflow.sh -c true
# this runs the entire workflow using the KLI and clears out the KERI 
# directories mentioned above
#
# Option 3: run with the KERI Agent (Mark I - deprecated) and the
#           clearing option
./workflow.sh -a true -c true
# this runs the workflow using the KLI only where absolutely necessary
# and then cURL requests to the Agent API to perform the majority of the 
# workflow
```

This is a great way to get you up and running. The entire workflow takes only 3 minutes for the KLI flow and 2.5 minutes for the Agent workflow. Reading through the BASH script may be the fastest way for you to learn how to use KERI if you are ready for that level of detail.

## ATHENA Network

The ATHENA network includes:
- three witnesses
- four controllers (with one custom controller, the Abydos Gatekeeper)
- four agents, one for each controller (Mark I Agent – deprecated)
- a credential caching server
- four distinct credential types.
- a webhook


You will, of course, set up all of the configuration files and keystores for each witness and controller as well as perform all of the OOBI configuration for the network to function. See below for an overview.

### Network Overview

{% image(path="/images/posts/07-abydos-athena-component-graph-1.png", class="diagram",
    op="fit_width",
    alt="ATHENA Network diagram") 
    %}
ATHENA Network diagram
{% end %}

### Controllers

There is one controller each for Richard, Elayne, Zaqiel, and Ramiel. You will write the configuration files for each controller and create and incept their keystores.

#### Custom Controller

The one custom controller is the one for Zaqiel, the Abydos Gatekeeper controller. This controller needs to be customized in order to make the access control decisions for Abydos. This includes custom credential validators and presentation response payload constructors.

### Witnesses

The same three witnesses are used for all of the controllers. Using the same witnesses for everything isn’t required and would be bad practice, though it simplifies the architecture for this post. You could easily use separate witnesses per controller with just a bit more tedious work. You will write the witness configuration files for each witness and set them up using the KLI.

### Credential Caching Server

The vLEI-server from the [vLEI repository](https://github.com/WebOfTrust/vLEI) is used as the credential caching server. It is a very simple HTTP fileserver that serves up ACDC schemas upon HTTP request. It would easily be implementable in any other language or tech stack if you needed it to be.

### Agents

The agents use the current agent code in the KERIpy repository, called the Mark I Agent. This agent code is now deprecated even though it is nearly fully fleshed out. The new Mark II Agent is being developed as a separate repository in the [KERIA repository](https://github.com/WebOfTrust/keria).

### Webhook

The webhook is a simple HTTP server built on the [ioflo/hio](https://github.com/ioflo/hio) library executed with `sally hook demo`. All it does is receive HTTP requests from the Abydos Gatekeeper custom controller, parse the JSON body, and print all of the headers and body values.

## Foundational Vocabulary and Theory

Next we cover some basic vocabulary and acronyms so you can make your way through this post. I recommend you review Nuttawut Kongsuwan’s KERI jargon [summary](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#de18) of the KERI whitepaper for an overview of the terminology in the space. You can always go to the [KERI Whitepaper](https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/KERI_WP_2.x.web.pdf) or my [KERI Mind Map blog post](https://www.kentbull.com/2022/06/05/keri-start/) as well for a list of documents central to the KERI space that will assist you in gaining a full understanding of the concepts and the terminology.

[KERI](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#48aa) – Key Event Receipt Infrastructure. A decentralized public key infrastructure for self certifying identifiers aiming at security as a top priority while following the design principle of “minimally sufficient means.” The foundation for the verifiable credentials technology called authentic chained data containers.

[AID](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#7344) -autonomic identifier. a self-certifying identifier that is the primary root of trust in KERI. AIDs are decentralized identifiers that do not rely on a blockchain.

[Controller](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#2627) – a node or an individual set of cryptographic keys, their key event logs (KELs), and associated data. Consists of a keystore directory, a set of autonomic identifiers (AIDs), key event logs, any credentials, transaction event logs, and out of band introductions.Also considered to be a running instance of a KERI controller software such as [KERIpy](https://github.com/WebOfTrust/keripy) (Python KERI controller) or [KERIox](https://github.com/THCLab/keriox) (Rust KERI controller).

[Witness](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#a998) – a trusted entity designated by another controller as a witness to anything that happens for an AID controlled by that controller. Each witness must provide witness receipts to each event in a key event log.

[KERIpy](https://github.com/WebOfTrust/keripy) – the Python implementation of KERI including the controller implementation. This Python implementation is considered authoritative as the KERI core team builds it along with Dr. Samuel M. Smith.

Agent – The HTTP API to a KERI controller.

OOBI – out of band introduction. The core discovery mechanism of KERI. Each OOBI defines a path of communication between controllers or a location where a resource like an ACDC schema resides. Think of this as something like a DNS entry or an iptables entry, or an address in an address book. If you have an OOBI for a controller or object you want to talk to then you can reach it over the internet.

ACDC – authentic chained data container; verifiable credential. The fundamental unit of data shared between KERI controllers meant for instrumenting business process workflows that use verifiable credentials.An append-only, graph-linked data structure that is secure and verifiable.

Edge – a link (or chain) between two ACDCs.

ACDC Schema – the detailed description of precisely what data is allowed to be placed into any given ACDC.

Credential Registry – where ACDCs are issued from.

Ricardian Contract – In ACDCs they are a way to link the legal system to specific ACDCs. This comes in the Rules section of ACDCs.

## Admittance Workflow

The purpose of all of this business about identifiers and credentials is to support a basic admittance workflow into Abydos, specifically the Osireion at the back of Abydos. In order to support this workflow we use verifiable credentials.

### Credentials Overview

For the explorers to be granted admittance into Abydos the admittance workflow must be followed:
1. The Wise Man must issue **TreasureHuntingJourney** credentials to each explorer wishing to go on a treasure hunt to Abydos.
2. Each explorer must issue a **JourneyMarkRequest** credential for the treasure hunt they want to embark on by pointing to a specific TreasureHuntingJourney credential in their request.
3. As the Wise Man approves each explorer he is to issue a **JourneyMark** as an approval of the explorer’s desire to embark on the journey. This approval should point to the specific request each explorer made for a specific journey.
4. Once a sufficient number of JourneyMark credentials are issued by the Wise Man and the journey threshold has been crossed he is to issue a **JourneyCharter** to each explorer signifying his approval of their journey.
5. The JourneyCharter credential is to be **presented by each explorer** to the Gatekeeper in order to be granted entrance into Abydos.


To understand this workflow we first cover the credential map itself, how to write a credential, and how to link each of the schemas together using the KASLCred tool.

### Visual workflows and use cases for credential issuance

Each of the credential issuance flows is described visually below.

#### 1 – TreasureHuntingJourney issuance flow

As a Wise Man (ATHENA official) I want to provide confidence to potential explorers so that I may attract them to go on a treasure hunting journey.

To do this the WiseMan issues the TreasureHuntingJourney credential.

{% image(path="/images/posts/07-abydos-whattowhom-thj.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### 2 – JourneyMarkRequest issuance flow

As an Explorer I want to indicate a serious interest on a specific TreasureHuntingJourney so that ATHENA will hold my spot.

To do this an Explorer (or Librarian) issues to the WiseMan a JourneyMarkRequest pointing to a specific TreasureHuntingJourney credential.

{% image(path="/images/posts/07-abydos-whattowhom-jmr.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### 3 – JourneyMark issuance flow

As a Wise Man I want to endorse a particular Explorer for a specific TreasureHuntingJourney so that the journey can be chartered once a threshold of party membership is reached.

To do this the Wise Man issues the JourneyMark to those Explorers whose JourneyMarkRequests he approves. The JourneyMark points to the specific JourneyMarkRequest submitted by an explorer.

{% image(path="/images/posts/07-abydos-whattowhom-jm.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### 4 – JourneyCharter issuance flow

As a Wise Man I want to charter a treasure hunting journey once a party membership threshold is reached so that the explorers can present this charter credential to the Gatekeeper and gain entrance into Abydos.

To do this the WiseMan issues the JourneyCharter credential to each Explorer going on the journey indicated in the TreasureHuntingJourney credential. The JourneyCharter points to both the JourneyMark and the TreasureHuntingJourney credential.

{% image(path="/images/posts/07-abydos-whattowhom-jc.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### 5 – JourneyCharter presentation flow

As an Abydos Gatekeeper I want to allow authorized Explorers into Abydos so that they can have a good time on a treasure hunting journey.

To do this the Explorers present the JourneyCharter credential to Zaqiel the Gatekeeper, Zaqiel then validates the credential, and makes an access control decision on whether to allow an Explorer in to Abydos.

{% image(path="/images/posts/07-abydos-whattowhom-present.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

## Credential Map

You decided to represent the secure proofs as four data containers (ACDC), or verifiable credentials (VC), with links between the different credentials as shown below. This graph shows the credentials and their links to each other, or edges. Each edge is named, as you will see later on in the schema map JSON file.

### ATHENA Credential Graph

{% img_left(path="/images/posts/07-abydos-athena-credential-graph.png", 
    width=300, height=200,
    alt="ATHENA credential graph", op="fit_width") 
    %}
ATHENA credential graph
{% end %}
- (1) **[TreasureHuntingJourney](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/treasure-hunting-journey.json)**: One for the Wise Man to describe the overall treasure hunting experience from start to end, treasure splits for hunting party members, the destination, and the like. This will be issued by the Wise Man to each potential explorer.
- (2) **[JourneyMarkRequest](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/journey-mark-request.json)**: One for a potential explorer to request to join the treasure hunting journey. This will be issued to the Wise Man by an explorer.
- (3) **[JourneyMark](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/journey-mark.json)**: One for the Wise man to issue to the accepted explorers so the Gatekeeper can know who the Wise Man has authorized for the journey.
- (4) **[JourneyCharter](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/journey-charter.json)**: One that the explorer party can use to prove they are authorized for their journey to all who care to know as well as for the Gatekeeper to verify that they were authorized as a party.


In order to understand the credential schemas we must first cover some background knowledge on why schemas in ACDC are constructed the way they are, the purpose they serve, and how to use them.

## ACDC Schema Writing

[JSON Schema](https://json-schema.org/specification.html) is the schema language ACDC schemas are expressed in. Beyond this schema language [self addressing data](https://www.ietf.org/archive/id/draft-ssmith-said-02.html#section-2.4-8) and [self addressing identifiers](https://www.ietf.org/archive/id/draft-ssmith-said-02.html#abstract) are the mechanism used to verify schemas. With **security first** as a design principle self addressing data combines well with JSON schema to provide a secure schema capability for the KERI and ACDC ecosystem.

### ACDC Credential Schemas with JSON Schema

For each ACDC we use there must be a schema defining the shape of the data in the container or credential. This schema must be verifiable and protected from schema revocation and schema malleability attacks. Using self addressing data (SAD) through the use of self addressing identifiers (SAIDs) makes it easy to accomplish both of these security objectives along with the business process objective of having a verifiable shape or envelope for our data.

#### What does an ACDC look like?

##### TreasureHuntingjourney credential issuance example JSON export

This is getting a little bit ahead of ourselves though a visual of what a credential looks like will help you understand where we are going. Take a quick look at what an issued JourneyMarkRequest looks like, how to get it, and then move on to the next section.

```json
{
  "v": "ACDC10JSON0001d9_",
  "d": "EEXZuecxP4Y3xZxvA_DtnrPX8nbSDPeGaMIxNKvLVENb",
  "i": "EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ",
  "ri": "EF3hGwYMK0r74qlSRbKw3_RzCrJ8D2Xmv5BlunrN3NC2",
  "s": "EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l",
  "a": {
    "d": "EPZSit7DYobEAv3WkVk2GBoBWrdglaHImi3_lnoc5fNl",
    "i": "EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q",
    "dt": "2023-03-21T15:39:14.560430+00:00",
    "destination": "Osireion",
    "treasureSplit": "50/50",
    "partyThreshold": 2,
    "journeyEndorser": "Ramiel"
  }
}
```

This JSON export of a credential is obtained with the KLI command `kli vc export` as shown below:

```bash
RICHARD_CRED_SAID=$(kli vc list --name explorer --alias richard --said --schema EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l)
kli vc export --name explorer --alias richard --said $RICHARD_CRED_SAID
```

This particular execution of the command assumes there is only one credential to be listed by the `kli vc list` command for the schema `EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l`, the schema identifier for the TreasureHuntingJourney schema. If there were multiple TreasureHuntingJourney credentials then there would be multiple SAIDs output by that command then you would pick one of them to be the argument to the `--said` parameter in order to export it’s JSON.

#### What is an ACDC schema made of?

##### TreasureHuntingjourney schema creation

The first schema to write is the JSON Schema for the **[TreasureHuntingJourney](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/treasure-hunting-journey.json)** credential that Richard and Elayne must receive from the Wise Man Ramiel in order to submit their treasure hunting request as a JourneyMarkRequest later. In order to write this schema we must first understand all of the elements that go into an ACDC schema.

###### Schema Parts

Each ACDC schema has five main parts:
1. Schema metadata (required)
2. Properties section (required)
3. Attributes section (required)
4. Edges section (if used)
5. Rules section (if used)


This is a collapsed view of the [**TreasureHuntingJourney** schema](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/treasure-hunting-journey.json).

{% image(path="/images/posts/07-abydos-thj-schema-collapsed.png", class="diagram",
    op="fit_width",
    alt="ACDC top-level view") 
    %}
ACDC top-level view
{% end %}

As you can see it has metadata, properties, attributes, and rules yet no edges. This is because this **TreasureHuntingJourney** ACDC does not extend nor connect with any other ACDC. This ACDC could appear as an edge in another ACDC though it does not have any edges itself.

Keep in mind that this instance of the TreasureHuntingJourney schema does not have any “$id” properties filled out yet. This tells you we are still writing the schema and are working from the template that will have the SAIDs added to, or “SAIDified.” We will do that step later after we finish writing the schema itself.

First we shall explain the attributes of the schema from outer to inner, top to bottom. Each of the attributes has a specific purpose we get into below.

###### SCHEMA description – top-layer

While ACDC schemas use the JSON Schema [specification](https://json-schema.org/specification.html) in most regards there is one exception where ACDC schemas repurpose the “$id” field. The top-most $id, $schema, title, description, type, credentialType, and version properties are about the ACDC schema itself. What is contained inside of the ACDC is described by what comes in the **properties** section which is further clarified by the **additionalProperties** and required attributes.
- **$id** : The self addressing identifier (SAID) of the overall schema. This can only be computed when all of the dependent SAIDs for all attribute (e), edge (e), and rule (r) blocks have been computed and embedded within the respective **$id** attributes of each block. This field is repurposed to provide a unique identifier of the schema instance. This is because in ACDCs all schemas are self addressing and so there is no separate document to retrieve such as in non-ACDC credentials, such as JSON-LD credentials, where the $id field contains a base URI fragment that needs to be resolved to a schema. Since the schema is keyed and stored by SAID rather than a Retrieval URI then there is little to no benefit from storing a retrieval URI in the $id section.See the [$id section](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-schema-id-field-label) of the ACDC specification for a more complete explanation. Example Value: EDiuqE2fD2MhrSpZhAzlZn8XLSJiHt1pyy1eEWB-l1qq

- This is distinct from the “v” version field described in the [Version String Field](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-version-string-field) section of the ACDC specification. This “version” field is the schema version. The ACDC “v” field specifies the serialization format version and the size of the serialized data type.
- Keep in mind these two fields are separate and are used for different purposes even though they have a similar name.
- Remembering that the *short name* occurs only in ACDC **instances** and that the *long name* occurs only in ACDC **schemas** is a good way to keep the concepts distinct in your mind.


With all of the metadata properties covered we next describe the **properties** section of the TreasureHuntingJourney credential where you describe the fields of your custom data that will be a part of the credential.

##### TreasureHuntingJourney Properties

Only a small amount of data is written inside of a TreasureHuntingJourney credential:

```json
{
  "destination": "Osireion",
  "treasureSplit": "50/50",
  "partyThreshold": 2,
  "journeyEndorser": "Ramiel"
}
```

The TreasureHuntingJourney credential is intended to provide information to prospective explorers about a journey they could embark on. This includes where the journey is headed, the destination, the proportion of treasure proceeds split amongst each journey party member, the minimum party threshold for the journey to be chartered, as well as the individual who chartered the journey.

The properties section of an ACDC schema is the location to include custom business-process data as shown above for the TreasureHuntingJourney credential.

###### Writing The Properties schema

Writing a schema to describe this small set of data requires a rather verbose description using JSON schema and though tedious is simple to understand once you get the hang of it.

In addition to this data there is a rules section “r” we will get into later when we talk about [Ricardian Contracts](https://en.wikipedia.org/wiki/Ricardian_contract) though for now we focus only on the data going into the credential, the “a” section.

Writing a properties section with JSON Schema to describe this requires both metadata and an attributes section.

{% image(path="/images/posts/07-abydos-thj-properties-collapsed.png", class="diagram",
    op="fit_width",
    alt="ACDC properties section") 
    %}
ACDC properties section
{% end %}

The ACDC functionality of KERI requires the presence of a number of metadata attributes in the **properties** section of a JSON Schema document as shown below. Each attribute is linked to the appropriate section in the ACDC specification. The overall field map description is provided here: [Top-Level Fields](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-top-level-fields).
- [v](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-version-string-field) – “Version”: This describes the version of the serialization format and size used to write the schema.
- **[d](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-said-self-addressing-identi)** – “SAID or identifier”: This provides a stable, universal, cryptographically verifiable and agile reference to the properties block.
- [u](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-uuid-universally-unique-ide) – “Universally Unique Identifier as a Nonce”: A salty nonce used to provide cryptographic security to the “d” field to protect against brute force attacks such as rainbow table attacks to guess the block’s contents. This field is optional.
- [i](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-aid-autonomic-identifier-an) – “Autonomic Identifier”: The identifier of the issuer of this ACDC.
- [ri](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-aid-autonomic-identifier-an) – “Registry Identifier (Derived Identifier)”: The registry is the credential registry that this particular ACDC was issued from. This is derived from the “i” field for the issuer who issued the ACDC. A controller can create and use one or more registries.
- [s](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-top-level-fields) – “Schema SAID”: The SAID of the schema to be used to validate this properties block.
- [a](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-attribute-section) – “Attribute map”: The list of attributes that defines this schema. This is where the JSON data from above gets placed into the credential.
- [r](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-rule-section) – “Rules section”: This is where any Ricardian Contracts (code as law) are stored in the credential.


Other than the metadata attributes the “a” and “r” properties, **attributes** and **rules**, are the more interesting parts. The “a” section is where you will describe the data properties that will be placed in your ACDC.

###### TreasureHuntingJourney Attributes Section

The attributes section can be one of two things as shown in the JSON schema, either a SAID or an inner block with its own properties.

```bash
...
"a": 
  "oneOf": [
    {
      "description": "Attributes block SAID",
      "type": "string"
    },
    {
      "$id": "EPfNU6jej4GGyprpakq6KCO9os9vp9jLIRcH1xJqVezj",
      "description": "Attributes block",
      "type": "object",
      "properties": { 
...
```

###### oneOf Operator and compact ACDC

The **oneOf** operator in JSON Schema means that the “a” section can be one of either of these two things. For ACDCs this means that the “a” block can be either the condensed version, which is the SAID of the “a” block, or it can be the entire contents of the “a” block. The condensed version is called a “[Compact ACDC](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-compact-acdc)” and can be seen as equivalent to the un-condensed, or un-compacted version of the ACDC.

We are not covering compact ACDCs in this article so we move on to the full or un-compacted ACDC.

###### Full or un-compacted ACDC

The full ACDC schema of the attributes block includes another set of metadata attributes, another identifier field labeled “$id”, and another properties section. JSON Schema as you can see is very verbose. Looking a level deeper we see the following set of attributes inside the “a” block:

{% image(path="/images/posts/07-abydos-image-3.png", class="diagram",
    op="fit_width",
    alt="full, un-compacted ACDC") 
    %}
full, un-compacted ACDC
{% end %}

Now this is starting to look like the JSON data we intend to put in the credential. Inside of the “a” block we see another “$id” field as well as other metadata fields from JSON Schema including the “description”, “type”, and “properties” fields. As those are mostly self explanatory we move on to the inner “properties” block.

Each property has a name, a description, and a type:

```bash
...
"properties": {
  "d": {
    "description": "Attributes block SAID",
    "type": "string"
  },
...
```

The “type” field must be one of the valid types from the list in [JSON Schema](https://json-schema.org/understanding-json-schema/reference/type.html#type-specific-keywords):
- [string](https://json-schema.org/understanding-json-schema/reference/string.html#string)
- [number](https://json-schema.org/understanding-json-schema/reference/numeric.html#number)
- [integer](https://json-schema.org/understanding-json-schema/reference/numeric.html#integer)
- [object](https://json-schema.org/understanding-json-schema/reference/object.html#object)
- [array](https://json-schema.org/understanding-json-schema/reference/array.html#array)
- [boolean](https://json-schema.org/understanding-json-schema/reference/boolean.html#boolean)
- [null](https://json-schema.org/understanding-json-schema/reference/null.html#null)


The “description” field is a schema-writer field to put a human-readable, concise definition of what the field contains.

###### Inner Properties block metadata fields

The inner properties block has a set of metadata fields that the ACDC code in KERI fills out for you including the “d” (SAID), “i” (issuee), and “dt” (date and time stamp) fields. They are required attributes in the top level of the inner properties block and look like the following in the JSON schemas where they appear:

```bash
...
"properties": {
  "d": {
    "description": "Attributes block SAID",
    "type": "string"
  },
  "i": {
    "description": "Issuee AID",
    "type": "string"
  },
  "dt": {
    "description": "Issuance date time",
    "type": "string",
    "format": "date-time"
  },
...
```

The “$id” field is a SAID of the “a” attributes block. This provides the “a” block with cryptographic protection and verifiability when signed by a KERI key.

###### Inner Properties Block Custom Fields

Following the required properties you define your own custom properties to be added to the ACDC. In the case of the TreasureHuntingJourney credential those properties include the “destination”, “treasureSplit”, “partyThreshold”, and “journeyEndorser” fields as follows:

```bash
..."properties": {  //  // Required properties  //  ...  "destination": {    "description": "The target location for this journey where the hunters will go.",    "type": "string"  },  "treasureSplit": {    "description": "The type of splits for this journey. 50/50, 25/25/25/25, and so forth. Must add up to 100.",    "type": "string"  },  "partyThreshold": {    "description": "The minimum party member threshold needed to charter this journey",    "type": "integer"  },  "journeyEndorser": {    "description": "The AID of the ATHENA inner circle member endorsing this treasure hunting journey.",    "type": "string"  }...
```

Each field has a description and a type. The validation of each field is up to the issuer or the verifier of the credential.

### The Edge Section

We will briefly cover the JourneyMarkRequest schema since we have not covered edge sections yet. The TreasureHuntingJourney credential has no edges so it isn’t a good example of a credential with an edge section.

#### JourneyMarkRequest Schema

As you can see the JourneyMarkRequest has an additional attribute, the “e” section. This is for edges, or chains between individual ACDCs.

{% image(path="/images/posts/07-abydos-image-12.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The edge section is very similar to an “attributes” section except it is specialized. It contains a “oneOf” operator similar to the “attributes” section since the edge section may also be represented by a SAID in the most compact version, or any compact version, of an ACDC.

{% image(path="/images/posts/07-abydos-image-13.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

Inside of the Edges block, the second item in this “oneOf” array, we see a set of common properties. $id, description, and type are what we have seen before, as is properties. Inside the properties block we find something slightly different.

{% image(path="/images/posts/07-abydos-image-14.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The “d” field is the SAID of this particular edges block and is computed from the other properties which in this case is just the “journey” property.

The “journey” property is the actual edge that points to the TreasureHuntingJourney credential (not the TreasureHintingJourney – typo).

The “n” field is the SAID of the far side credential, the TreasureHuntingJourney.

The “s” field is the SAID of the ACDC schema for the TreasureHuntingJourney.

Defining an edge section like this in a schema is the foundation of what is needed to add in an edge between these two credentials later during the credential issuance process.

### The Other Schemas

With this nearly exhaustive explanation of one of the four schemas complete and a small amount on the JourneyMarkRequest edge we will leave the writing or reading of the other three of the four schemas up to the reader. This includes the JourneyMarkRequest, JourneyMark, and JourneyCharter. You may find the ATHENA version of the other schemas in the athena[/schemas](https://github.com/TetraVeda/abydos-tutorial/tree/master/athena/schemas) directory. They will be described briefly below at a high level. You can apply the schema writing concepts from above to all of the other schemas.

Just glance over the following pictures to get a sense of what is in each schema and them move on to the credential linking section afterwards.

#### JourneyMark Schema

Has the following properties:

{% image(path="/images/posts/07-abydos-image-15.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

And edge properties for “request”

{% image(path="/images/posts/07-abydos-image-16.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### JourneyCharter Schema

Has the following properties:

{% image(path="/images/posts/07-abydos-image-17.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

And the following edges. Something different for the JourneyCharter schema is that it links to two edges, both “mark” and “journey”:

{% image(path="/images/posts/07-abydos-image-18.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

### Linking Schemas

One concept we have not yet covered is how to link schemas together. We cover this next in the Schema Linking and Verification section via the use of the [KASLCred utility](https://pypi.org/project/kaslcred/).

## Schema Linking and Verification

Linking schemas with [self addressing data](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#section-2.4) is part of the way ACDCs provide security, specifically end-verifiability, of credentials. For data to be self addressing it must contain an identifier that is computed from the contents of the data, a digest. Self addressing data protects against security exploits like schema malleability and schema revocation attacks while also being an easy way to verify the contents of the data. These self addressing identifiers are placed in the “[$id](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html#name-said-self-addressing-identi)” attributes of the various blocks in ACDC schemas.

Here is an example self addressing identifier for the TreasureHuntingJourney ACDC shown above: `EEXZuecxP4Y3xZxvA_DtnrPX8nbSDPeGaMIxNKvLVENb`

### Self Addressing Data (SAD) and Self Addressing Identifiers (SAIDs)

The self addressing identifier (SAID) is like a key in a key-value pair and the data is like the value except that the identifier computation algorithm allows for the SAID to be embedded in the data. This requires a two pass process, one to calculate the identifier, and the second to embed it within the data. Verifying self addressing data also requires a two-pass verification function, one pass to remove the embedded identifier, and the second to verify the data against the SAID. To fully understand the self addressing identifier concept refer to the [IETF Draft Specification for Self-Addressing IDentifier (SAID)](https://weboftrust.github.io/ietf-said/draft-ssmith-said.html) by Samuel smith.

### Schema Evaluation to compute SAIDs

Using self addressing identifiers (SAIDs) with a graph of schemas means that the credential schemas must be evaluated in a deterministic order and have their “$id” properties populated from the inner-most or top-most schema to the outer-most or bottom-most schema. In this sense the outer-most schema points to the innermost schema and the bottom-most schema points to the top-most schema as in the **ATHENA Credential Graph** diagram above and reproduced below for reference. This proper order requires computing a dependency graph of the schemas based on their edges (dependencies) in order to facilitate generating identifiers on the inner-most (or top-most) schemas and then each layer outward to the outermost (or bottom-most) schemas.

{% img_left(path="/images/posts/07-abydos-athena-credential-graph.png", 
    width=300, height=200,
    alt="athena credential graph", op="fit_width") 
    %}
athena credential graph
{% end %}

### Graph Evaluation Order

In our case our graph must be generated in the following order:
1. TreasureHuntingJourney (root node, no dependents)
2. JourneyMarkRequest (dependent on TreasureHuntingJourney)
3. JourneyMark (dependent on JourneyMarkRequest)
4. JourneyCharter (dependent on both JourneyMark and TreasureHuntingJourney)


Your options here are to either do this by hand with the `kli saidify` command and manually editing all of the schema files or you can use the code in the [generate.py](https://github.com/WebOfTrust/vLEI/blob/dev/src/vlei/generate.py) file from the [WebOfTrust/vLEI](https://github.com/WebOfTrust/vLEI) repository as a guide. Alternatively you could use the [KASLCred](https://pypi.org/project/kaslcred/) Python package to do all of the linking for you. For this post we will use KASLCred in the interest of time.

### KASLCred

While the intricacies of parsing ACDC schemas and adding SAIDs in the “$id” attributes are interesting, also called “saidifying” the schemas, in the interest of time we will use the KASLCred utility written to do all the saidifying for you. You can go read the Python code of the [TetraVeda/kaslcred](https://github.com/TetraVeda/kaslcred) repository if you want to see how it was done. It’s only a few hundred lines of code.

#### KASLCred Schema Map

Using KASLCred to evaluate your schema graph involves writing a schema map file expressing the set of credentials to be evaluated in a JSON file. The “athena/schemas/athena-schema-map.json” file from the tutorial repository is reproduced below for reference:

```json
{
  "schemas": [
    {
      "schemaName": "JourneyCharter",
      "schemaFilePath": "journey-charter.json",
      "dependencies": ["JourneyMark", "TreasureHuntingJourney"],
      "edgeName": ""
    },
    {
      "schemaName": "JourneyMarkRequest",
      "schemaFilePath": "journey-mark-request.json",
      "dependencies": ["TreasureHuntingJourney"],
      "edgeName": "request"
    },
    {
      "schemaName": "TreasureHuntingJourney",
      "schemaFilePath": "treasure-hunting-journey.json",
      "dependencies": [],
      "edgeName": "journey"
    },
    {
      "schemaName": "JourneyMark",
      "schemaFilePath": "journey-mark.json",
      "dependencies": ["JourneyMarkRequest"],
      "edgeName": "mark"
    }
  ]
}
```

#### Evaluation Order

As you can see this list of schemas is out of order. The first item in the list, JourneyCharter, should be evaluated last. And the third item in the list, TreasureHuntingJourney, should be evaluated first. This is no problem for KASLCred as it reads the **dependencies** array and reorders the schemas to be evaluated in the order where the least dependent item (with no dependencies) is evaluated first and then the next most dependent item is evaluated after all of its dependencies is evaluated and so forth for all dependencies.

Using KASLCred is as simple as a Pip Install and a `python -m` invocation once you have Libsodium and KERI installed:

```bash
# Install KERI and Libsodium as noted in the installation instructions above
pip install kaslcred==0.0.8
# Set ABYDOS_REPO_DIR to where you cloned the abydos-tutorial repo.
# Usage: python -m kaslcred [schemas dir] [results dir] [schema map file]
python -m kaslcred \
  ${ABYDOS_REPO_DIR}/athena/schemas \
  ${ABYDOS_REPO_DIR}/athena/saidified_schemas \
  ${ABYDOS_REPO_DIR}/athena/schemas/athena-schema-map.json
```

As long as all of the file names in the `schemaFilePath` properties in the schema map JSON file exist then you will end up with a set of SAIDified schemas ready to use.

### Quick overview of the other schemas

Review the JSON contents of the abydos-tutorial/athena/credential_data directory to get a sense for what kind of business process we are building on top of these schemas. We reproduce them here to illustrate their purpose.

#### JourneyMarkRequest data – for Richard

```json
{
  "requester": {
    "firstName": "Richard",
    "lastName": "Ayris",
    "nickname": "Dunkie"
  },
  "desiredPartySize": 2,
  "desiredSplit": 50.00
}
```

As you can see here this credential looks a lot like an account registration HTTP REST API request. You can really do anything with ACDC credentials. It also indicates preferences to the ATHENA explorer matchmaking engine with the desired party size and the desired split.

#### JourneyMark data – for Richard

```json
{
  "journeyDestination": "Osireion",
  "gatekeeper": "Zaqiel",
  "negotiatedSplit": 50.00
}
```

This data indicates where Richard will be going and other details of the journey. Remember that the JourneyMark has a graph edge pointing to the JourneyMarkRequest which means credential consumers can also resolve data from both the JourneyMarkRequest and the TreasureHuntingJourney credentials through those links.

#### JourneyCharter data – for Richard

```json
{
  "partySize": 2,
  "authorizerName": "Ramiel"
}
```

As the final data packet sent as an ACDC this credential indicates what each explorer stands to expect on the chartered journey such as how many members they will be traveling with and who officially authorized their journey.

## Schemas Review

Now that we have written the necessary schemas for ATHENA and have SAIDified them with KASLCred we are ready to use them in configuring our trust network components as well as to issue and verify credentials.

Next we configure our trust network to cache these schemas with the vLEI caching server implementation (vLEI-server).

## Network Configuration

Configuring a KERI network involves setting up the following components:
- ACDC Schema Caching
- Witnesses
- Controllers
- Agents (if you are using them)
- Performing Out Of Band Introductions (OOBIs) between controllers and objects


Our setup will involve the following additional components
- A customized controller used for the Abydos Gatekeeper to decide whether or not explorers can be permitted to enter Abydos. It will be configured to trust one issuer, the Wise Man, that will be used to check the issuance of the JourneyCharter credentials against.
- A webhook the customized controller communicates with to tie credential presentation and revocation events into a custom business logic layer.


Here is the network component diagram again for reference:

{% image(path="/images/posts/07-abydos-athena-component-graph.png", class="diagram",
    op="fit_width",
    alt="Abydos network component diagram") 
    %}
Abydos network component diagram
{% end %}

Schema caching is first item to be set up as all of the controllers depend on it. You could set up witnesses first as well. It is up to you.

### Schema Caching

In order to successfully receive, validate, and handle an ACDC each KERI controller using that ACDC must be able to resolve and load the schema for that ACDC. The schemas are resolved based on the SAID of the schema using an out of band introduction (OOBI) that indicates the location the schema is stored at and can be retrieved from. The controller may either be configured with the location of this schema on startup in the controller bootstrap JSON configuration file or be configured at runtime through usage of the OOBI resolution mechanism.

#### vLEI-server

{% img_left(path="/images/posts/07-abydos-athena-caching-server.png", 
    width=300, height=200,
    alt="vLEI-server", op="fit_width") 
    %}
vLEI-server
{% end %}

The vLEI-server provides a convenient implementation of a simple caching server that meets our needs. All the vLEI caching server (vLEI-server) does is make the schemas available as JSON objects as responses to HTTP requests. It could very easily be rewritten in your favorite language if you wanted a fun, easy challenge. (Hint, write it in Rust, the KERI community will be happy with you.)

Once you have the vLEI repository cloned and the vLEI-server installed as described in the installation instructions above you can run the vLEI-server with a command similar to the following:

```bash
# Change directories to the abydos-tutorial repository directory.
# Set ATHENA_DIR to be your abydos-tutorial/athena directory.
vLEI-server \
  -s "${ATHENA_DIR}"/saidified_schemas \
  -c "${ATHENA_DIR}"/cache/acdc \
  -o "${ATHENA_DIR}"/cache/oobis
```

This will give you output similar to the following:

```bash
caching schema EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l
caching schema ELc8tMg_hhsAPfVbjUBBC-giEy5440oSb9EzFBZdAxHD
caching schema EBEefH4LNQswHSrXanb-3GbjCZK7I_UCL6BdD-zwJ4my
caching schema EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr
```

If you have anything in the `cache/acdc` or `cache/oobis` directories then those will be shown in the log output as well.

With the ACDC schemas successfully being cached the next step is to deploy the witness network which requires writing witness configuration files.

## Witness Network

Our witness network consists of three of the witnesses used in the KERI Demonstration Witness network meaning we use the same cryptographic key salts, keystore names, and alias names.

This means we need a configuration file for each of the witnesses. The document symbols you see on the preceding diagram show the filenames for each of the witnesses represented by a gray circle.

To set up the witness network we must do the following things:
1. Write witness configuration files ensuring the witnesses listen for incoming traffic on a set of TCP and HTTP ports.
2. Create the keystores for the witnesses using the KERI command line interface, the KLI, via `kli init` subcommand.
3. Start witnesses using the KLI via the `keri witness start` subcommand.


For the ATHENA witness network we create three separate witnesses as shown above on the following ports:
- `wan`: TCP 5632 HTTP 5642
- `wes`: TCP 5633 HTTP 5643
- `wil`: TCP 5634 HTTP 5644


### Building your own, non-demo witnesses and agents

In order for you to really understand the KERI KLI we do not want you to have to rely on any demo commands such as `kli witness demo` or `kli agent vlei` so we show you here how to use the keystore and witness commands directly.

First we address the writing of witness configuration files.

### Witness Configuration files

Each witness on startup reads a witness configuration file. This file includes properties such as TCP and HTTP ports to listen on and any OOBIs to resolve at startup. See below for a witness configuration file example. The diagram shows the specific witness configuration file used for each specific witness.

{% image(path="/images/posts/07-abydos-athena-witconfig.png", class="diagram",
    op="fit_width",
    alt="witness network with config files") 
    %}
witness network with config files
{% end %}

#### Location and Directory Structure

JSON configuration files for KERI witnesses must be placed inside a particular directory structure. This directory structure is as follows`CONFIG_ROOT/keri/cf/main`

{% image(path="/images/posts/07-abydos-image-4.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The reason for this is that the KERI [Configer module](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/configing.py#L29) uses the root path of “keri/cf” and the sub path of “main” as see in the [Configer.__init__](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/configing.py#L50) function. You can configure the CONFIG_ROOT directory path by customizing the `--config-dir` property though the `keri/cf/main` directory path will be created for you inside of the CONFIG_ROOT directory if they do not already exist.

Then, inside this `CONFIG_ROOT/keri/cf/main` directory you must place your witness configuration JSON files.

#### Witness Configuration File Contents

The contents of a witness configuration file look as follows:

```json
{
  "wan": {
    "dt": "2022-01-20T12:57:59.823350+00:00",
    "curls": ["tcp://127.0.0.1:5632/", "http://127.0.0.1:5642/"]
  },
  "dt": "2022-01-20T12:57:59.823350+00:00",
  "iurls": [
  ]
}
```

There must be a key in the configuration object named the same as the value passed into the `--alias [alias]` property for the subsequent `kli witness start` command. This applies to KERIpy, the Python implementation of KERI and the reference implementation of the KERI and ACDC specification.

Refer specifically to KERIpy [habbing.pyL971](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/habbing.py#L971) in the `reconfigure` function to see how the configuration file must have an appropriately named key. Keep in mind that this function documentation states that “Config file is meant to be read only at init not changed by app at run time” so you should make any changes to this configuration before the `kli witness start` commands are run since any changes during run time will not be recognized until the witness is stopped and then started again.

The values inside the configuration file are as follows. Sub-properties such as the “dt” property inside of “wan” are indicated with the dot notation.
- “wan”: The named configuration block property for a given witness. “wan.dt”: The date-time stamp when the configuration was created.
- “wan.curls”: Controller URLs. This is the list of URLs that indicate the protocol and ports for the witness to listen on. You can see both TCP port 5632 and HTTP port 5642 in the above example.


Other properties that are not shown above yet can be presented include:
- “durls”: Data URLs. Another section for OOBI URLs to process. Typically you put credential OOBIs in this section when you want the witness to resolve them on startup. You could put these URLs in the “iurls” section as well.
- “wurls”: Well-known OOBIs to resolve on startup. TODO clarify difference between and purpose of each type of URL here.


With this explanation of how to write a witness configuration file you now can go make the other two witness configuration files for “wil” and “wes,” or you can pull them from the tutorial repo: [athena/conf/keri/cf/main/*.json](https://github.com/TetraVeda/abydos-tutorial/tree/master/athena/conf/keri/cf/main)

### Witness Create Command: kli init

Next we must create the keystores. We will end up with the following three keystores:

{% image(path="/images/posts/07-abydos-athena-witkeystores.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

We use the `kli init` command to create each of the keystores.

A witness requires a keystore to be created first in order to start the witness later. Do this with a command like the following:

```bash
kli init --name wan --salt 0AB3YW5uLXRoZS13aXRuZXNz --nopasscode \
    --config-dir "${CONFIG_DIR}" \
    --config-file main/wan-witness
```

You can see you must name the witness and pass in a configuration directory and file. The configuration file must have the “main/” path prefix on it since the underlying `Configer` class defaults the “base” path property to be “main” as seen on [Configer:L50 here](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/configing.py#L50). You don’t need to add the “.json” suffix since KERIpy will do that for you. The filename alone is sufficient.

Running this command will create output similar to the following:

```bash
KERI Keystore created at: /Users/myuser/.keri/ks/wan
KERI Database created at: /Users/myuser/.keri/db/wan
KERI Credential Store created at: /Users/myuser/.keri/reg/wan
```

#### Keystore Passcodes

Here I am not using any passcodes though you are free to modify the example to use them. The passcode is an additional layer of security used to unlock a keystore. Using a passcode would look like adding in a passcode argument like so: `--passcode DoB26Fj4x9LboAFWJra17O`

#### For Each Witness

Write a configuration file for each of the three witnesses, wan, wil, and wes. You can use the `create_witnesses` function in the `workflow.sh` script as a guide. Place each of the JSON configuration files in your configuration directory with the appropriate directory suffix like so:`$CONFIG_DIR/keri/cf/main`

#### Cryptographic Salts (seeds)

The salt used here is the same as both what is used in the Abydos Tutorial witness set as well as what is used for the demo command `kli witness demo` that starts six witnesses for you. This is insecure. For production you should use separate salts. Using the same salt means the keys generated from the salt will be the same. So you may conclude that protecting your salt is as important as protecting your keys themselves similar to protecting your Bitcoin or Ethereum seed phrase and private keys.

Whatever you do make sure your seeds do not get in the hands of people who shouldn’t have them and be sure to use highly secure deployment mechanisms to inject seeds into your witness, agent, and controller deployments.

Moving on, next we start the witness.

### Witness Start Command: kli witness start

Starting a witness is very simple yet it does depend on code recently added to KERIpy to support the `--config-dir` and `--config-file` arguments so make sure you install the latest version of KERIpy from source with `***python -m pip install -e ./***` from the KERIpy repository root directory. This will give you at least KERIpy 1.0.0 release locally whether or not the release has been pushed to PyPi or GitHub yet.

```bash
kli witness start --name wan --alias wan \
  -T ${WAN_WITNESS_TCP_PORT} \
  -H ${WAN_WITNESS_HTTP_PORT} \
  --config-dir "${CONFIG_DIR}" \
  --config-file wan-witness
```

Ports and the alias name are a few additional configuration options here. Remember that the “alias” property must match the name used in the witness configuration file you wrote earlier in which you defined the TCP and HTTP endpoints for your witness to listen on. The other arguments are as they seem, -T for the TCP port and -H for the HTTP port the witness will listen on.

Run this command for each of the three witnesses wan, wil, and wes. You can use whatever ports you’d like though the ATHENA ports used are shown in the [variables section](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L109) near the top of the `workflow.sh` script.

Running this command successfully will provide output similar to the following:

```bash
Witness wan : BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha
```

You can either do the commands by hand yourself or you can use the `workflow.sh` script to do it for you. It is recommended to get some practice yourself with the commands in order to learn them though you can start by reading the `start_witnesses` function in the `workflow.sh` script to see how the Abydos Tutorial starts the ATHENA witnesses. Be aware that BASH variables are used heavily throughout the script so you will need to refer often to the variables section at the top of the script in order to fully understand what each of the commands are doing.

Once you have all of the witnesses set up and started your next task is to set up the KERI controllers for Richard, Elayne, Ramiel, and Zaqiel. This involves writing a bootstrap configuration file which we get into next.

## KERI Controller Setup

Creating and setting up controllers can happen in one of two ways, with the KERI command line, the KLI, or with the KERI Agent API (called the Mark I Agent, currently deprecated). The difference between the two is that the KLI method does not start a running process or daemon while the Agent runs as a daemon. An important note on the existing Agent API in KERIpy is that Agent API is now deprecated in favor of the new multi-tenant Agent API being written as a separate deployable called [KERIA](https://github.com/WebOfTrust/keria) (called the Mark II Agent).

While the existing Agent API is deprecated it is nonetheless useful to see how the KLI compares to using REST requests in the Agent API. You get an idea of how you might write your own API on top of a KERI controller or how you might contribute to the agent implementations. Once the Mark II Agent API is ready this article will be updated with the new API.

There are three general steps to setting up a controller:
1. Write the controller configuration file.
2. Initialize the controller’s keystore
3. Create the first keypair for the controller, called a prefix, by performing an inception event. This involves using an alias to label that particular prefix.


The KLI flow is shorter than the Agent flow though they both accomplish the same objective. You may follow along in the `workflow.sh` script with the code for setting up each of the four controllers in the following functions:
- For the Agent flow `start_agents` starts up all needed agents.
- `read_witness_prefixes_and_configure` updates the controller and agent bootstrap configuration files.
- `make_keystores_and_incept_kli` and `make_keystores_and_incept_agent` perform the keystore creation and inception events using the KLI or Agent API, respectively.


### Four Controllers

There are four controllers to be created, one each for Richard, Elayne, Ramiel, and Zaqiel. This involves writing configuration files with OOBIs for witnesses, and credential schemas, initializing controller keystores, performing the inception events, and, in the case of using agents, starting the KERI Agent for each controller. Below is a simplified diagram of the controllers and the out of OOBI connections that must be made between the controllers to support credential issuance and presentation.

{% image(path="/images/posts/07-abydos-athena-ctlr-net.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

### Controller Bootstrap Configuration File

A bootstrap configuration file instructs a KERI controller regarding which OOBI URLs it needs to resolve on initial startup. This is one of two ways a KERIpy controller can resolve OOBIs, either at bootstrap time or after a controller has been initialized. The bootstrap configuration file includes both a list of controller OOBIs and a list of credential schema OOBIs. The list of controller OOBIs is usually a list of OOBIs for each witnesses that will be used to perform the inception event though it can be expanded to other controllers beyond the witness pool.

When first starting a controller it reads in the configuration file specified by the `--config-file` argument to either the `kli init` or `kli agent start` commands. Beware that the configuration file is not re-read on subsequent startups. If you need a controller to resolve more OOBIs after initial bootstrapping then use the `kli oobi resolve` command or equivalent Agent REST request.

#### Example Config

The example config file from the ATHENA network shows three controller OOBI URLs in “iurls” and four credential schema OOBI URLs in “durls”. Starting an agent also reads in a similar configuration file with OOBIs for the agent to resolve during initialization. Agents are covered in a later section.

Once you have all three prefixes for the witnesses you can make OOBIs and write them into configuration files for both KLI and Agent modes:
- **controller-oobi-bootstrap.json** for the KLI mode Full path: abydos-tutorial/athena/conf/keri/cf/controller-oobi-bootstrap.json

- Full path: abydos-tutorial/athena/conf/keri/cf/agent-oobi-bootstrap.json


If you look into each of these two configuration files you will find their contents to be identical and will look very similar to the below:

```json
{
  "dt": "2022-01-20T12:57:59.823350+00:00",
  "iurls": [
    "http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "http://127.0.0.1:5644/oobi/BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "durls": [
    "http://127.0.0.1:7723/oobi/EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l",
    "http://127.0.0.1:7723/oobi/ELc8tMg_hhsAPfVbjUBBC-giEy5440oSb9EzFBZdAxHD",
    "http://127.0.0.1:7723/oobi/EBEefH4LNQswHSrXanb-3GbjCZK7I_UCL6BdD-zwJ4my",
    "http://127.0.0.1:7723/oobi/EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr"
  ]
}
```

Next we learn how to write each of the two OOBI lists for both the “iurls” and the “durls” sections of the configuration files.

#### IURLS: Writing controller OOBIs for the “iurls” section

An OOBI URLs uses the following URL scheme:“`[protocol]://[host]:[port]/oobi/[prefix]`“

The protocol is typically “http” though may be “https” or “tcp”. Host and port will be specific to your deployment. Creating a prefix involves many steps as shown in a later section. Without jumping ahead you can use the prefixes automatically created during the `kli witness start` command you ran earlier. This created a set of keystores which also created a set of prefixes for each witness. You can get the prefixes you created with the `kli status` command as shown below.

##### Prefixes with kli status

The prefix is the prefix of the first key in a sequence of KERI key creation events (establishment events). You can get the prefix with the `kli status command` as shown in the [read_witness_prefixes_and_configure](http://update_config_with_witness_oobis) function in `workflow.sh`. Here is an example using `kli status` and AWK to store witness wan’s prefix in the `WAN_PREFIX` variable:

```bash
WAN_PREFIX=$(kli status --name wan --alias wan | awk '/Identifier:/ {print $2}')
# Then do echo $WAN_PREFIX to see the value of the prefix
echo $WAN_PREFIX
```

The [update_config_with_witness_oobis](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L352) function does this for you in the `workflow.sh` script. Once you finish writing your witness OOBIs to the configuration file your “iurls” section should look like the following:

```json
{
  "dt": "2022-01-20T12:57:59.823350+00:00",
  "iurls": [
    "http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "http://127.0.0.1:5644/oobi/BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "durls": [
    ...
  ]
}
```

##### GENERATING KERI DIDs

The AID prefix is the same prefix that is added into the KERI decentralized identifier (DID) generation function when you run `kli did generate`. To turn the prefix into a did you do as follows:`did:[prefix]`You simply place the AID prefix after the characters “did:” and you have a KERI decentralized identifier.For example, using the first OOBI below, the corresponding DID would be:`did:keri:BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha`

Next we write the “durls” section for credential schema OOBIs. This enables controllers to recognize ACDC schemas by their SAID.

#### DURLS: WRITING credential schema OOBI URLs for the “durls” section

The four OOBIs in the “durls” are created by getting the “$id” properties from each of the schemas you created during the schema writing and linking step above with KASLCred. The scheme for data OOBIs is as follows:`[protocol]://[caching_server_host]:[caching_server_port]/oobi/[schema_said]`

For example, “`EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr`” is the `[schema_said]` property for the OOBI url for the JourneyCharter OOBI URL. This. is pulled directly from the schema in abydos-tutorial/athena/saidified_schemas/JourneyCharter__EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr.json

{% image(path="/images/posts/07-abydos-image-5.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The resulting OOBI URL looks like the following:

`"http://127.0.0.1:7723/oobi/EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr"`

In this instance the caching server host and port correspond to the `vLEI-server` instance ATHENA defines in the `workflow.sh` script, or `http://127.0.0.1` and `7723`, respectively.

Once you have the OOBI URLs set up then write them into both the controller-oobi-bootstrap.json file and the “agent-oobi-bootstrap.json” files in the “$CONFIG_DIR/keri/cf” configuration directory. In the Abydos Tutorial the location “`abydos-tutorial/athena/conf/keri/cf`” is the $CONFIG_DIR configuration directory.

These files should look somewhat like the following once you are finished:

```json
{
  "dt": "2022-01-20T12:57:59.823350+00:00",
  "iurls": [
    "http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "http://127.0.0.1:5644/oobi/BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "durls": [
    "http://127.0.0.1:7723/oobi/EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l",
    "http://127.0.0.1:7723/oobi/ELc8tMg_hhsAPfVbjUBBC-giEy5440oSb9EzFBZdAxHD",
    "http://127.0.0.1:7723/oobi/EBEefH4LNQswHSrXanb-3GbjCZK7I_UCL6BdD-zwJ4my",
    "http://127.0.0.1:7723/oobi/EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr"
  ]
}
```

Now that you have the controller and agent bootstrap configuration files set up we show how to start up agents with the KLI.

## KERI Agents

### KERI Agent Startup – Mark I agent (deprecated)

Creating and handling a controller with the KERI Mark I agent involves a four step process:

{% img_left(path="/images/posts/07-abydos-athena-agent.png", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}
1. Start the agent with `kli agent start`
2. POST to /boot to initialize the keystore
3. PUT to /boot to unlock the keystore
4. POST to /ids/${ALIAS} to perform the inception event


There are four agents to be set up for the ATHENA network, one each for Richard, Elayne, Ramiel, and Zaqiel as shown below:

{% image(path="/images/posts/07-abydos-athena-agents.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### KERI Controller Agent Start (Mark I Agent – deprecated)

You will notice that the `--insecure` option is passed to the agent start command. This is because the first draft of a KERI Agent, the Mark I agent, has a partially complete HTTP signature signing scheme that currently does not function and thus must be disabled with the `--insecure` option. Without disabling this option you will not be able to communicate at all with a Mark I agent. The next version of KERI Agents is [KERIA](https://github.com/WebOfTrust/keria), the Mark II agent, and this has a fully complete HTTP signature signing scheme intended to work with [Signify](https://github.com/WebOfTrust/signify-ts), a TypeScript edge signing library for KERI.

The `--path` option declares a directory to be served up at the `$AGENT_URL/static` HTTP endpoint though isn’t important for this tutorial. It was originally used to host Swagger API documentation for the Agent API.

All of the other configuration options are relatively straightforward.
- `--config-dir` is the directory the agent bootstrap file will be located in Full path: abydos-tutorial/athena/conf/keri/cf

- **agent-oobi-bootstrap.json** for the Agent mode
- Full path: abydos-tutorial/athena/conf/keri/cf/agent-oobi-bootstrap.json


```bash
kli agent start --insecure \
  --admin-http-port ${EXPLORER_AGENT_HTTP_PORT} \
  --tcp ${EXPLORER_AGENT_TCP_PORT} \
  --config-dir ${CONFIG_DIR} \
  --config-file ${AGENT_CONFIG_FILENAME} \
  --path ${ATHENA_DIR}/agent_static
```

Start up an Agent for each controller Richard, Elayne, Ramiel, and Zaqiel. The [`start_agents`](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L242) function in `workflow.sh` shows one way to do this.

#### Switching between the KLI and the KERI Agent API (Mark I – deprecated)

For the remainder of this tutorial we switch back and forth between the KERI command line interface, the KLI, and the KERI Agent HTTP API, the Mark I version in KERIpy, in order to present a broad understanding of both the ways you can interact with the KERI specification and the work that has been completed in the space. This will provide an opportunity to understand both how the command line interaction looks as well as how a REST API addressing a similar purpose feels and looks.

Each task will be completed first with the KLI and then with the KERI Agent API. The KLI is always up to date with the latest and greatest code in KERIpy. The KERI Agent API will likely eventually be removed from KERIpy once the Mark II agent in [KERIA](https://github.com/WebOfTrust/keria) is finished. At present the KERI Agent API in the KERIpy repository is now deprecated and is referred to as the Mark I agent.

For our next task of creating keystores we start first with the KERI Command Line Interface (KLI) to initialize the keystores for all of the participants in this journey including the ATHENA officials and the two explorers. After showing how to accomplish this task with the KLI we switch to perform the same task with the Agent API.

##### Use one or the other for keystores and inception

Make sure to either use the KLI or the Agent API to perform keystore creation and inception, not both. You only need to create the keystores once whether your use the KLI or the Agent API.

If you don’t know which one you want or need for your scenario then start with the KLI.

## KERI Keystores and Inception

### KERI Controller Keystore Create – KLI INIT

There are four keystores to initialize. We will show here the commands to initialize one of the keystores. After performing this command you can initialize all of the other keystores yourself with similar commands for each keystore or use the `make_keystores_and_incept_kli` function in the `workflow.sh` script.

{% image(path="/images/posts/07-abydos-athena-ctlr-keystores.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### Salts

The cryptographic salt is one of the arguments to `kli init`. This salt is what all of the private keys are derived from so it should be protected as if you were protecting your personal identity documents or your bank account. You can get your own salt with the `kli salt` command:

```bash
$ kli salt
0ACPpGmPWX9LqRlUCOLH3qk2
```

You can use the salt in any `--salt` argument via the KLI or a `"salt"` property of a Agent request.

#### Command

The other arguments are straightforward, nopasscode, config-dir, and config-file. `--nopasscode` indicates that there is no passcode to access this keystore.

The `--config-dir` property indicates the directory in which to look for configuration files. Inside this configuration directory the file name from the `--config-file` option is used to find and select the controller bootstrap configuration file that will be used to initialize this keystore. This option does not need to have a file extension if your configuration file already ends with the “.json” extension.

```bash
kli init --name ${EXPLORER_KEYSTORE} --salt "${EXPLORER_SALT}" \
  --nopasscode \
  --config-dir "${CONFIG_DIR}" \
  --config-file "${CONTROLLER_BOOTSTRAP_FILE}"
```

This command sets up all of the [Lightning Memory Mapped Database](http://www.lmdb.tech/doc/) instances used by the keystore. These are located in the $HOME/.keri directory.

### KERI Controller AID (Prefix) Create and Incept – KLI INCEPT

Now the keystore is ready to have a specific key pair created and an inception event performed. This will begin the key event log (KEL) for that key pair. Key pairs are referred to with a label called an “alias.” This alias label is intended to be more human-friendly and is used throughout both the KLI and the Agent API to refer to the key pair.

#### Alias vs Prefix

Aliases are used to refer to key pairs since prefixes, the internal reference to key pairs, are not human friendly. For example, an alias can be a word or set of words like “richard” and a prefix looks like the following: `"EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q"`. As you can see you won’t be remembering many prefixes which is why the alias system is so helpful.

#### Steps to Inception

Performing an inception event, once the keystore is initialized, requires the following two steps:
1. Write an inception configuration file.
2. Execute the inception command whether from the command line or the Agent REST API For the KLI you will provide the inception configuration file as an argument.
3. For the Agent API you will provide the inception configuration as JSON in the body of the inception HTTP request.


Four inception events need to be performed on each of the four keystores. Each inception event begins a [key event log](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#7c43) (KEL), as shown below:

{% image(path="/images/posts/07-abydos-athena-ctlr-incept.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

First we write the inception configuration file.

#### Writing a Controller Inception Configuration File

Each attribute of a controller inception configuration file is important. The initial set of witnesses, keypair counts, key thresholds, transferability, and threshold of accountable duplicity are all defined in this file.

The following is the inception configuration file used for each of the controllers we create for the Abydos journey:

```json
{
  "transferable": true,
  "wits": [
    "BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "toad": 3,
  "icount": 1,
  "ncount": 1,
  "isith": "1",
  "nsith": "1"
}
```

##### Witness configuration file attributes

Now an explanation of each attribute, which may also be seen in [keripy/src/keri/app/habbing.pyL873](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/habbing.py#L888) and in [keripy/src/keri/app/cli/common/incepting.pyL9](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/app/cli/common/incepting.py#L9):
- “transferable” – [Transferability](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#7344) refers to whether a KERI autonomic identifier (AID) can have its keys rotated. Technically this refers to having what is called “control authority” changed, control over the private keypairs linked to a KERI AID. When true this means the AID is a transferable AID.When false this means the AID is non-transferable.
- “wits”: Witnesses – A list of prefixes identifying the witnesses to include signatures for in the inception event in this controller’s key event log.
- “toad”: Threshold of accountable duplicity – The number of agreeing witnesses receipts at or above which the controller accepts accountability for signed key events. Accountability is not accepted by the controller for key events with a number of signatures below this threshold.
- “icount”: Inception signature count – The number of keys used to sign the inception event.
- “ncount”: Number of rotation signatures count – The number of keys to be used on the next key rotation event. Note that not all pre-rotated keys committed to in the inception event as described [Reserve Rotation](https://weboftrust.github.io/ietf-keri/draft-ssmith-keri.html#name-reserve-rotation).
- “isith”: Signing threshold for the inception event – The number of key event receipts from distinct witnesses the Inception Event must have in order to be authoritative.
- “nsith”: Signing threshold for the next rotation event – The number of key event receipts from distinct witnesses the next Rotation Event must have in order to be authoritative.


##### Writing your configuration

For all except the “wits” section you can copy and paste the configuration file as-is. The “wits” section contains the AID prefixes that will be specific to your witnesses. If you are using the `workflow.sh` script with precisely the same salts, witness keystore names, and witness AID aliases, then you can use the configuration from above verbatim.

Place all of your configuration in a JSON file in a configuration directory you will remember the location of that you can reference when passing in the `--file` argument to the `kli incept` command below. In the Abydos Tutorial repo this path is “`abydos-tutorial/conf/inception-config.json`“.

#### Performing Inception – KLI INCEPT

Incepting an identifier consists of a set of steps that use keys generated by the user to create, sign, get receipts for, and store an inception event in the key event log (KEL) of a controller. The conceptual steps are outlined next.
1. Creating the number of cryptographic key pairs as indicated in the “icount” configuration property.
2. Creating the number of pre-rotated cryptographic key pairs as indicated in the “ncount” configuration property.
3. Creating a valid, properly formatted inception event signed by enough keys to satisfy the “icount” number.
4. Adding that inception event to the controller’s key event log.
5. Sending the inception event to each witness in set of witnesses configured in the inception configuration you wrote above. The set of witnesses is defined in the configuration file passed to the `--file` argument.
6. Receiving key event receipts (KERs) of the inception event from each witness.
7. Adding the key event receipts from the witnesses to the key event receipt log (KERL) for the controller.


All of these steps are performed for you when you execute the `kli incept` command. The below example from `workflow.sh` shows the inception of Richard’s AID:

```bash
kli incept --name ${EXPLORER_KEYSTORE} --alias ${EXPLORER_ALIAS} \
  --file "${CONTROLLER_INCEPTION_CONFIG_FILE}"
```

Repeat this procedure for each Elayne, Ramiel, and Zaqiel with the keystore and alias names indicated in the BASH variables at the top of the `workflow.sh` script in the [Abydos Tutorial repository](https://github.com/TetraVeda/abydos-tutorial).

Next, for those who are using agents, we show how to use the KERI Mark I Agent to perform keystore setup and inception.

### KERI Controller Keystore Create – Agent (Mark I – deprecated)

Keep in mind that you only need to create the keystores once whether you create the keystores with the KLI or the Agent API. Only proceed to create the keystores and perform inception with the Agent API if you have not already done so with the KLI.

Setting up a KERI controller keystore and AID with an agent involves the following three REST requests:
1. POST to `/boot` to initialize the keystore This is similar to the `kli init` command.
2. PUT to `/boot` to unlock the keystore This does not have a direct analogue with the KLI though the `--passcode` argument used with `kli init`, `kli incept`, and many other KLI commands serves a similar purpose.
3. POST to `/ids/${ALIAS}` to perform the inception event This is similar to the `kli incept` command.


Data in the request bodies for each of these requests is very similar to the command line argument data for each of the corresponding commands using the KLI.

#### curl Common Headers, arguments, and pipes

##### Headers

The “Accept: */*” and “Content-Type: application/json” HTTP headers are common to all of the commands used in the `workflow.sh` scripts.

##### Arguments

The “-s” argument to cURL is the “silent” option and ensures that only the response body of the target HTTP request is sent to the standard output channel.

The backslash [“\” escape character](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Escape-Character) is often used in BASH scripts to make commands more readable by enabling splitting of the commands across multiple lines.

##### Pipes

jq: Heavy usage of the [jq](https://stedolan.github.io/jq/) program, a lightweight and flexible command-line JSON parser, is made throughout this tutorial. Most often the results of a cURL command will be piped to the jq program and then the jq program will select an attribute from the response body using a selector expression like `'.["msg"]'` in the example below.

tr: Frequently the [tr](https://en.wikipedia.org/wiki/Tr_(Unix)) program is used to trim double quotes off of the JSON output returned from the jq program. As seen in the example below tr is used with the `-d` flag to delete all tokens of the specified set from input. This is needed since often the value of a cURL request needs to be placed in a BASH variable and the extra quotes returned along with the JSON body are problematic to deal with an so it is easier to remove them altogether.

Both jq and tr are often used together in a sequence as shown below.

```bash
...(more code causing JSON to be sent to standard out)... \
| jq '.["msg"]' | tr -d '"'
```

### KERI Controller Keystore Create – Agent

This operation is a POST to the `/boot` endpoint which creates a keystore where the body is a JSON object with the value placed in the “name” property of the JSON body used to name the keystore. The “salt” value is the cryptographic salt used as the cryptographic seed to derive all key pairs in the keystore. You can use the `kli salt` command to generate a new salt.

The `${EXPLORER_AGENT_URL}` is define on [Line 127 of workflow.sh](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L127) and looks like this:[http://127.0.0.1:5620](http://127.0.0.1:5620)

```bash
curl -s -X POST "${EXPLORER_AGENT_URL}/boot" \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  --data "{\"name\": \"${EXPLORER_KEYSTORE}\",
           \"salt\": \"${EXPLORER_SALT}\"}" \
  | jq '.["msg"]' | tr -d '"'
```

As you can see the “name” and “salt” options are similar to the similarly named arguments to KLI init. The difference here is that they are in the body of the POST request as JSON. The result is piped to “jq” and then “tr” to extract and print the result message to the terminal.

Next the keystore must be unlocked.

### KERI Controller Keystore Unlock – Agent

Unlocking a keystore is a PUT to the `/boot` endpoint using the same JSON body as was POSTed to initialize the keystore. This is used to permit a particular Agent to work with a specified keystore. This is useful because an agent may have been restarted after initializing the keystore and so the Agent must establish an authenticated connection to the keystore.

Bear in mind that the salt is being sent across the wire here so this should only ever be done locally between a wallet client and a local agent. This is not secure to do over the internet.

```bash
curl -s -X PUT "${EXPLORER_AGENT_URL}/boot" \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  --data "{\"name\": \"${EXPLORER_KEYSTORE}\",
           \"salt\": \"${EXPLORER_SALT}\"}" \
  | jq '.["msg"]' | tr -d '"'
```

Next the keystore is ready to have an AID, also known as a prefix, created and incepted.

### KERI Controller AID (Prefix) Create and Incept – Agent

Creating an AID requires submitting a POST request to the `/ids/{alias}` path of a controller agent. The body of the request is the same controller inception configuration file you wrote earlier when performing `kli incept`. The file is reproduced below for reference:

#### Inception Configuration File

```json
{
  "transferable": true,
  "wits": [
    "BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha",
    "BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM",
    "BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX"
  ],
  "toad": 3,
  "icount": 1,
  "ncount": 1,
  "isith": "1",
  "nsith": "1"
}
```

The contents of this file are placed in the body of the HTTP request as submitted below:

```bash
curl -s -X POST "${EXPLORER_AGENT_URL}/ids/${EXPLORER_ALIAS}" \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  --data @${CONTROLLER_INCEPTION_CONFIG_FILE} \
  | jq '.["d"]' | tr -d '"'
```

The jq and tr pipes here select the AID prefix that comes in the response body of the request. You can see on [Line 471 of workflow.sh](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L471) this sequence is used to capture Richard’s prefix and store it in the `RICHARD_PREFIX` bash variable.

### Keystore and Inception Review

Once you get the hang of creating keystores and performing inception events then you will see the similarities between the KLI and the Agent API. When you have finished creating all of the keystores you need, one each for the Explorer, Librarian, Wise Man, and Gatekeeper, then you are ready to move on to the next step of connecting all of the agents together using out of band introductions, or OOBIs.

## Out of Band Introductions (OOBIs) Between Controllers

Routes in a routing table or DNS entries are internet discovery system analogues to help understand the purpose out of band introductions. ATHENA requires a number of components in the Abydos network to talk to each other and thus an OOBI must be created or “resolved” for each connection between components.

{% image(path="/images/posts/07-abydos-athena-oobis-all.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

You can think of an OOBI as an address with a role for one controller to contact either another controller or a KERI resource such as a credential schema. In order for one controller to be able to issue a credential to another controller it must know where that other controller is. The role of an OOBI determines what to use the OOBI for, such as a witness, a credential schema, or just a contact.

For example, in the diagram above, in order for Ramiel to issue the TreasureHuntingJourney credential to Richard he must have an OOBI that shows the location of Richard that includes a witness for Richard. This witness OOBI is necessary so Ramiel can retrieve and verify Richard’s KEL from the witness and ensure it matches up to the AID prefix included in the OOBI URL. The “wan” witness is used in this case.

### OOBIs described

Out of band introductions (OOBIs) are the discovery mechanism for KERI controllers and KERI objects such as ACDC Schemas or cached ACDCs. OOBIs tell one controller how to reach another controller or object and are identified both by an AID prefix as well as an alias label.

A witness OOBI looks like the following example:

```bash
"http://127.0.0.1:5642/oobi/EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q/witness/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha"
```

As you can see there are two KERI AID prefixes, one for the controller and another for the witness. The template for this OOBI URL is as follows:

```bash
[proto]://[wit host]:[wit port]/oobi/[controller prefix]/[role]/[wit prefix]
```

The terminology “out-of-band” here refers to the fact that the general purpose internet infrastructure like IP routers, gateways, and DNS servers, are not part of KERI and are thus out-of-band with respect to KERI. The primary advantage to using this general purpose infrastructure for service discovery means KERI doesn’t have to provide it’s own discovery mechanism since it can piggy back on the existing internet infrastructure.

Discovery over the internet includes endpoint discovery of witnesses, watchers, and other network participants including [jurors](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#285f) and [judges](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0#28f1) (not yet implemented). According to the [Out-Of-Band-Introduction (OOBI) Protocol](https://weboftrust.github.io/ietf-oobi/draft-ssmith-oobi.html) specification:

In the specification the usage of the terms “endpoint” or “service endpoint” just mean the URL to use to access some KERI resource.

#### OOBIs must be verified

The above quote draws attention to the fact that that OOBI URLs themselves are not trustable and must be verified. What does this mean? What is being verified? Since OOBI URLs merely point to a controller’s AID (prefix) the OOBI must be used to retrieve the full key event log from that controller referenced by the prefix. This key event log is then verified using the witness identified by the witness prefix at the end of the OOBI, in the case of a witness OOBI. If the controller proves control over the key referenced by the KEL then the verifier of the OOBI can then trust they have securely discovered the controller referenced by the OOBI.

#### OOBIs may have roles

This is an OOBI that shows a witness for a given prefix. This type of OOBI includes a “witness” role and a witness prefix. This witness prefix corresponds to the witness host and port earlier in the OOBI URL. This witness will have a full key event log (KEL) of all of that controller’s key events and thus can be used to verify that the AID prefix of the controller’s KEL verifies against the controller prefix included in the OOBI URL. This verification process is a part of what is known as “resolving” an OOBI.

See [kering.py:L23](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/kering.py#L23) for a full list of other OOBI roles including controller, witness, registrar, watcher, judge, juror, peer, mailbox, and agent, at present.

### Richard to Ramiel OOBIs – pairwise exchange

The following modified OOBI diagram shows greater detail of what the OOBIs exchanged between Richard and Ramiel look like. Since both Ramiel and Richard exchange OOBIs with each other this is called a pairwise OOBI exchange.

{% image(path="/images/posts/07-abydos-athena-oobis-richard-ramiel.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

Since both witness OOBIs use the “wan” witness to resolve key state they have a secondary arrow pointing to wan in addition to their primary arrow to indicate this relationship.

Each controller wanting to communicate with another controller must resolve an OOBI with the target controller describing where the controller is located over the internet because otherwise the two controllers would not know where the other is. Resolving OOBIs in a pairwise manner enables the source controller to send messages to that destination controller.

In the case of Ramiel an OOBI must be resolved for both Elayne and Richard in order for Ramiel to issue the TreasureHuntingJourney credential to them because Ramiel must know where Elayne and Richard are over the internet to be able to communicate with them. Similarly, Elayne and Richard both must resolve an OOBI that describes where Ramiel is located over the internet. All of these introductions are performed in both the [make_introductions_kli](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L525) and [make_introductions_agent](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L581) functions in the `workflow.sh` script.

### Abydos OOBIs

The journey to Abydos requires that each controller is known for a certain reason. These reasons may include:
1. Credential Issuance: an issuing controller must have an OOBI for each issuee’s controller receiving a credential. This is the case when Ramiel needs to issue the TreasureHuntingJourney credential to both Richard and Elayne. Likewise, Richard must know how to contact Ramiel in order to issue him the JourneyMarkRequest credential.
2. Credential Presentation: a credential holding controller must have the OOBI for the target node they will be sending a credential presentation to so that the presentation can reach the intended destination. This is the case when Richard and Elayne want to send the JourneyCharter credential to Zaqiel in order to be permitted entrance into Abydos.


These OOBI introductions are the controller-to-controller OOBIs needed during network operation. Keep in mind that there were OOBIs resolved in network setup and configuration for each of the witnesses as well as the four credential schemas used.

Use the commands below to perform all of the needed OOBI introductions as indicated on the graphic below.

{% image(path="/images/posts/07-abydos-athena-oobis-witness.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

#### Controller Introductions – KLI

Using the KLI to resolve a witness OOBI for a controller involves
- specifying the target keystore with the `--name` argument,
- setting an OOBI alias with the `--oobi-alias` property and
- passing in a witness OOBI to the `--oobi` argument as shown below.


```bash
kli oobi resolve --name ${WISEMAN_KEYSTORE} \
  --oobi-alias ${EXPLORER_ALIAS} \
  --oobi ${WAN_WITNESS_URL}/oobi/${RICHARD_PREFIX}/witness/${WAN_PREFIX}
```

An OOBI resolution is a very simple operation and produces output similar to the following:

```bash
http://127.0.0.1:5642/oobi/EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q/witness/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha resolved
```

#### Controller Introductions – Agent (Mark I – deprecated)

Using the Agent API to resolve an OOBI is very similar to the KLI. All the properties are passed in through the HTTP request body as a JSON object as shown below.

```bash
curl -s -X POST "${WISEMAN_AGENT_URL}/oobi" \
  -H "accept: */*" \
  -H "Content-Type: application/json" \
  -d "{\"oobialias\": \"${EXPLORER_ALIAS}\",       
       \"url\":\"${WAN_WITNESS_URL}/oobi/${RICHARD_PREFIX}/witness/${WAN_PREFIX}\"}" \
  | jq
```

This should result in a 202 HTTP status code when successful.

Once you have performed all OOBI creations for the controllers as depicted in the diagram above then move on to creating credential registries.

## Credential Registries

{% image(path="/images/posts/07-abydos-athena-registry-ramiel.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

In order to issue credentials from one controller to the next a credential registry must exist in both the issuing controller’s keystore to create and issue the credential from as well as in the receiving controller’s (issuee’s) keystore to receive the issued credential. This requires adding a new concept to manage the credential registry as well as any credentials issued from the registry. Enter the transaction event log [transaction event log](https://github.com/trustoverip/acdc/wiki/transaction-event-log) (TEL).

### Transaction Event Logs (TELs)

A transaction event log is a log of all events occurring for a verifiable credential or a credential registry that manages verifiable credentials. All TEL events are anchored in a KEL in either `ixn` (interaction) or `rot` (rotation) events. This is the foundation enabling a verifiable credential protocol to be built on top of KERI. This protocol is known as the authentic chained data container, or ACDC protocol. See the [ACDC specification](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html) for a complete description of this protocol.

Two further sub-protocols within the ACDC paradigm further elaborate on creating registries and handling the verifiable credential lifecycle. These are the Public Transaction Event Logs (PTEL) and the Issuance and Presentation Exchange (IPEX) protocols.

### Public Transaction Event Logs (PTEL) protocol

PTEL has two major parts the first being the details of using transaction event logs (TELs) to connect ACDCs to key event logs (KELs) and the second being the handling of events in the lifecycle of a credential including issuance and revocation.

There are two types of TELs:
1. A [management TEL](https://weboftrust.github.io/ietf-ptel/draft-pfeairheller-ptel.html#name-management-tel) for managing an ACDC (credential) registry and tracks the list of Registrars that will act as Backers for individual TELs for each verifiable credential (VC).
2. A [Verifiable Credential TEL](https://weboftrust.github.io/ietf-ptel/draft-pfeairheller-ptel.html#name-verifiable-credential-tels) (VC TEL) for managing the issued and revoked state of an individual ACDC. The VC TEL also contains a reference to its corresponding management TEL. There are only two events in a VC TEL which are the issuance event and then a subsequent revocation event, if ever revoked.


PTEL describes the details of transaction event logs. The following introduction from the [PTEL specification](https://weboftrust.github.io/ietf-ptel/draft-pfeairheller-ptel.html) summarizes well the purpose of transaction event logs:

### Issuance and Presentation Exchange (IPEX) protocol

Building upon the PTEL specification IPEX describes what happens between credential registries. This protocol defines the disclosure workflow events used to share information between the Discloser and the Disclosee.

### Registry Creation

{% img_right(path="/images/posts/07-abydos-blue-credential.jpg", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}

A registry creation event occurs in a TEL and has the type `vcp`. See the “[Verifiable Credential Registry](https://weboftrust.github.io/ietf-ptel/draft-pfeairheller-ptel.html#name-verifiable-credential-regis)” section of the PTEL specification for a complete description of TEL event types. To avoid overburdening the diagram below the individual `ixn` events for each credential issuance are not included in the KEL nor are the `iss` events included in the TELs. The blue credential graphic to the right is used on successive versions of the diagram to represent credential issuances.

The below diagram shows the registry creation event as an interaction event in the KEL of each controller as well as the corresponding TEL created to manage the creation of each credential issued by or to each controller.

{% image(path="/images/posts/07-abydos-athena-registries.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The IN column in a TEL indicates credentials the controller has received.The OUT column in a TEL indicates credentials the controller has issued.

Both of these columns start out empty and will be filled as credentials are issued.

#### Registry Creation – KLI

All that is needed to create a registry is:
- a keystore name with the `--name` argument to select the keystore to create the registry inside of,
- an alias with the `--alias` argument to select the AID to use to issue credentials from that registry,
- and the name of the registry with the `--registry-name` argument.


```bash
kli vc registry incept --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --registry-name ${EXPLORER_REGISTRY}
```

This creates a registry named by the `--registry-name` argument value with a management TEL. The registry’s management TEL will be used to keep track of the backers for each credential.

Additional arguments accepted by the command not shown above include:
- `--nonce`: unique random value to seed the credential registry.
- `--no-backers`: boolean to not allow setting up backers different from the anchoring KEL.
- `--establishment-only`: boolean to only allow establishment events for the anchoring of events in this registry.
- `--backers`: the new set of backers different from the anchoring KEL witnesses to be used when setting backers for credentials from this registry. Can appear multiple times.
- `--base`, `--alias`, and `--passcode` like the `kli init` and `kli incept` commands.


There is no TOAD (threshold of accountable duplicity) argument for the KLI since as of March 2023 it has not been needed nor has the external backer feature been used or tested in production. Nonetheless a default TOAD is computed based on the length of the backer list sent in as you can see on [Line 95 of eventing.py](https://github.com/WebOfTrust/keripy/blob/446eca9a694166e24416221ec49736cb071cd790/src/keri/vdr/eventing.py#L95). If you need this feature then be sure to open an issue on the KERIpy GitHub repo and attend a [community dev meeting](https://github.com/WebOfTrust/keri#meetings) (every other Tuesday at 8 AM UTC-6 DST / UTC-7 Non-DST).

#### Registry Creation – Agent (Mark I – deprecated)

Creating a registry using the Agent API is similar to the KLI except that the values passed as arguments to `kli vc registry incept` are instead located in the body as a JSON object.

```bash
curl -s -X POST "${EXPLORER_AGENT_URL}/registries" \
  -H "accept: */*" \
  -H "Content-Type: application/json" \
  -d "{\"alias\":\"${EXPLORER_ALIAS}\",
       \"baks\": [],
       \"estOnly\":false,
       \"name\":\"${EXPLORER_REGISTRY}\",
       \"noBackers\":true,
       \"toad\":0}" \
  | jq
```

Once again here the output is piped to “jq” though this is only to make the response easy on the eyes.

Next we move to one of the most exciting parts of this journey, the issuing of credentials!

## Credential Issuance

{% img_left(path="/images/posts/07-abydos-blue-credential.jpg", 
    width=300, height=200,
    alt="", op="fit_width") 
    %}
Image
{% end %}

Value generation from the entire decentralized identity space centers around verifiable credentials. Wrap some data attributes in a container defined by a schema and you now have a verifiable bearer instrument that is a part of your digital reputation. A credential is the fundamental unit, or currency, of reputation.

### Name – Why ACDC?

Now where does the name authentic chained data container come from and how did it get attached to the verifiable credential space? Three points make this clear:
1. Since ACDC-style credentials are verifiable as **authentic** using the KERI decentralized key management infrastructure (DKMI),
2. since ACDC-style credentials can be easily **chained** as a directed, acyclic graph (DAG),
3. and since the schema of an ACDC creates a shape or **container** for **data**,


you can see how those concepts combined to create the name (1) authentic (2) chained (3) data container, or ACDC.

### Purpose

The introductory quote from the ACDC spec sums up well the purpose of the ACDC protocol:

Verifiable authorizations, permissions, rights, and credentials that include a full provenance chain anchored in key event logs make KERI an elegant way to capture and unlock reputation value as well as serve as a robust solution for identity and access management systems.

To learn in-depth about all the particulars for ACDCs read the [Authentic Chained Data Containers (ACDC) IETF draft specification](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html) by Dr. Samuel M. Smith.

With this brief interlude on the name and purpose of ACDCs out of the way next we dive into the specific credentials we will issue for the journey to Abydos.

### Credentials for Abydos

Multiple different credentials are issued during different parts of the overall ATHENA trust workflow in the journey to Abydos. The following graphic shows a small blue icon representing each issued (OUT) or received (IN) credential.

{% image(path="/images/posts/07-abydos-credentials-all.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The four credentials that will be issued over the course of the journey include:
1. TreasureHuntingJourney: this is issued by the Wise Man to each of the explorers, Richard and Elayne, to tell them about a potential journey they could go on. It represents a commitment by ATHENA to a potential explorer as an offer of a journey they could embark upon.
2. JourneyMarkRequest: this is issued by an explorer to the Wise Man as a commitment to join the party for a particular TreasureHuntingJourney. Both Richard and Elayne issue this to Ramiel.
3. JourneyMark: this is issued by the Wise Man to an explorer as an acknowledgment that ATHENA accepted that explorer onto the requested TreasureHuntingJourney. This is issued by Ramiel to both Richard and Elayne.
4. JourneyCharter: once the party threshold for a journey is reached (threshold logic not yet implemented) the Wise Man issues a JourneyCharter credential to each party member who will be going on the journey. This includes Richard and Elayne.


On to our first credential!

### Issue TreasureHuntingJourney Credential – KLI

The TreasureHuntingJourney must be issued to both Richard and Elayne as depicted in the graphic below.

{% image(path="/images/posts/07-abydos-credentials-thj.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

As you see Ramiel has both of these credentials in the OUT column to indicate he has issued these credentials and both Richard and Elayne have them in the IN column indicating they have received these credentials.

The following command shows issuance of this credential to Richard.

```bash
kli vc issue --name ${WISEMAN_KEYSTORE} \
  --alias ${WISEMAN_ALIAS} \
  --registry-name ${WISEMAN_REGISTRY} \
  --schema "${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}" \
  --recipient "${RICHARD_PREFIX}" \
  --data @"${ATHENA_DIR}"/credential_data/osireion-treasure-hunting-journey.json
```

This shows issuance of a credential:
- `--name`: from the Wise Man’s keystore
- `--alias`: using the Ramiel AID
- `--schema`: using the schema SAID of the TreasureHuntingJourney schema.
- `--recipient`: The recipient is Richard
- `--data`: and the data sent in is specific to the credential issued to Richard.


The only arguments we haven’t used yet for this command are as follows:
- `--edges`: This is for adding the edge links to other credentials, the “chain” part of authentic chained data containers. We will see this in use for the next credential we issue, the JourneyMarkRequest.
- `--rules`: This is where the Ricardian Contracts come in to ACDCs. The TreasureHuntingJourney credential did not use the `--rules` option since the rules were baked in to the schema definition. However, the JourneyMarkRequest schema is different and uses rules passed in.


#### Richard’s TreasureHuntingJourneyCredential Data

```json
{
  "destination": "Osireion",
  "treasureSplit": "50/50",
  "partyThreshold": 2,
  "journeyEndorser": "Ramiel"
}
```

This is the data located in the [abydos-tutorial/athena/credential_data/osireion-treasure-hunting-journey.json](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/credential_data/osireion-treasure-hunting-journey.json) file. It is used for both Richard’s and Elayne’s credential.

Once the credential is issued using `kli vc issue` it can take a few moments to arrive so the `kli vc list ... --poll` command is useful to wait for the credential to arrive. Make sure to list the credentials because polling causes internal KERI messages to be delivered.

```bash
kli vc list --name ${EXPLORER_KEYSTORE} --alias ${EXPLORER_ALIAS} --poll
```

The output of this command looks as follows:

```bash
Checking mailboxes for any received credentials......
Current received credentials for richard (EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q):
Credential #1: EE0MvGRafksqpzyCYXm6tfaiKYR9LpUsV8YGD8KSMRIS
    Type: Treasure Hunting Journey
    Status: Issued ✔
    Issued by EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ
    Issued on 2023-03-27T20:23:48.536543+00:00
```

As long as you see the `Status: Issued ✔` then you can know you have issued the credential correctly.

### Issue TreasureHuntingJourney Credential – Agent (Mark I – deprecated)

Issuing credentials with the Agent is very similar to the command line except all of the data arguments are added as JSON body attributes as shown below.

```bash
JOURNEY_DATA=$(cat "${ATHENA_DIR}"/credential_data/osireion-treasure-hunting-journey.json)
curl -s -X POST "${WISEMAN_AGENT_URL}/credentials/${WISEMAN_ALIAS}" \
  -H "accept: application/json" \
  -H "Content-Type: application/json" \
  -d "{\"credentialData\":${JOURNEY_DATA},
       \"recipient\":\"${RICHARD_PREFIX}\",
       \"registry\":\"${WISEMAN_REGISTRY}\",
       \"schema\":\"${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}\"}" \
  | jq '.["d"]' | tr -d '"'
```

Once again I pipe the response to jq and tr to select the SAID of the credential which returns in the “d” field. This is useful to store the SAID in a BASH variable (JOURNEY_DATA) for use later in the `workflow.sh` script.

#### Get SAID of TreasureHuntingJourney credential issued

Another way to get the SAID of the TreasureHuntingJourney credential is by using a GET request to the `/credentials/`${ALIAS} endpoint.

```bash
EXPLORER_JOURNEY_CRED_SAID=$(curl -s \
  -X GET "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}?type=received&schema=${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}" \
  | jq '.[0] | .sad.d' | tr -d '"')
```

Use this GET request if you need to read in the SAID of a particular AID. The pipe combination to selects the “d” attribute from the first credential in the list (“.[0]”) returned and stores it in the EXPLORER_JOURNEY_CRED_SAID variable.

## Advanced Credential Issuance – with Edges and Rules

### ACDCs with Edges

The preceding credential issuance of TreasureHuntingJourney credentials was interesting yet it did not include one of the critical features of ACDCs: edges. Since a graph of ACDCs is connected by edges then knowing how to use an edge to connect, or “chain,” credentials together enables you to represent your domain’s credential data as a graph data structure, specifically a directed acyclic graph, which is just an append-only data structure. The JourneyMarkRequest credential includes an edge that points to the TreasureHuntingJourney credential as shown in the graph below.

{% image(path="/images/posts/07-abydos-athena-creds-thj-to-jmr.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

The edge from the JourneyMarkRequest is labeled as “journey”. This is represented in the schema for the JourneyMarkRequest as well as in the schema map JSON file that KASLCred reads in. Edges can be traversed to resolve, inspect, and validate the credential the edge points to.

### ACDC Rules – Ricardian Contracts

We also show an alternate way to specify rules with the `--rules` option to the KLI and the “rules” property of the JSON object sent to the Agent.

### Issue JourneyMarkRequest Credential – KLI

Once the JourneyMarkRequest Credentials are issued your network will be in the following state:

{% image(path="/images/posts/07-abydos-credentials-jmr.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

Both Richard and Elayne will have issued JourneyMarkRequest credentials to Ramiel and these credentials will have an edge pointing back to the TreasureHuntingJourney credential Ramiel issued to them.

A few steps are involved in issuing the JourneyMarkRequest credential due to the fact that it points to another credential, the TreasureHuntingJourney credential. These steps include:
1. Prepare the TreasureHuntingJourney edge
2. Saidify the TreasureHuntingJourney edge
3. Include the TreasureHuntingJourney edge inside the JourneyMarkRequest during issuance.


#### Prepare TresureHuntingJourney Edge

A few steps are required to prepare the TreasureHuntingJourney edge for inclusion in the JourneyMarkRequest credential. All of these steps are used to write a JSON file with the edge data that will be included to either the `--data` argument to `kli vc issue` with the KLI or embedded within the “source” attribute of the JSON object POSTed to `"/credentials/{alias}`” with the Agent.

##### JQ filters

JQ filters are useful to write values to a JQ object which eventually will become JSON. See the “Object Construction” section of the “[Types and Values](https://stedolan.github.io/jq/manual/#TypesandValues)” section of the JQ documentation for a full explanation. In the below code, as well as in the [issue_treasurehuntingjourney_credentials_agent](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L697) function in `workflow.sh` the following JQ object is used to construct an edge:

```json
{  
  d: \"\", 
  journey: {
    n: ., 
    s: \"${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}\"
  }
}
```

The BASH variable “TREASURE_HUNTING_JOURNEY_SCHEMA_SAID” is added in before the `echo` statement writes that JQ object to the file “JOURNEY_EDGE_FILTER”. This SAID is what it seems, the SAID of the TreasureHuntingJourney ACDC Schema. This is written into the JQ filter object.

JQ is then used to add the “EXPLORER_JOURNEY_SAID” into the “n” field of the JQ filter using the period “.” expression with the input piped into `jq` to output the resulting edge JSON.

```bash
JOURNEY_EDGE_FILTER=${ATHENA_DIR}/credential_edges/richard-journey-edge-filter.jq
RICHARD_JOURNEY_EDGE=${ATHENA_DIR}/credential_edges/richard-journey-edge.json
echo "{d: \"\", journey: {n: ., s: \"${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}\"}}" >"${JOURNEY_EDGE_FILTER}"
EXPLORER_JOURNEY_SAID=$(kli vc list --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --said \
  --schema "${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}")
echo \""${EXPLORER_JOURNEY_SAID}"\" \
  | jq -f "${JOURNEY_EDGE_FILTER}" >"${RICHARD_JOURNEY_EDGE}"
kli saidify --file "${RICHARD_JOURNEY_EDGE}"
# Add SAID to the JourneyMarkRequest rules
kli saidify --file "${ATHENA_DIR}"/credential_rules/journey-mark-request-rules.json
```

#### SAIDify TreasureHuntingJourneyEdge

Finally after writing the edge JSON the file is SAIDified with `kli saidify`. This populates the “d” attribute of the JSON with a self addressing identifier, or SAID. A self addressing identifier, according to the [spec](https://weboftrust.github.io/ietf-said/draft-ssmith-said.html#section-1-2), is:

This essentially means the value of the “d” field is computed from the value of everything else in the JSON object and as such is a value derived from everything else in the JSON object, thus self-addressing. Read the spec to learn about the specialized derivation function required to verify a SAID.

The following is an example of a complete TreasureHuntingJourney edge:

```json
{
  "d": "EGRPqBFL0xM0CPeTyc487pfO-MtR5UlYub0S-Rpyx6nm",
  "journey": {
    "n": "EE0MvGRafksqpzyCYXm6tfaiKYR9LpUsV8YGD8KSMRIS",
    "s": "EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l"
  }
}
```

Once the SAIDified file is created then the “journey” edge is ready to be incorporated into the issuance of the JourneyMarkRequest credential.

#### Issue JourneyMarkRequest credential with edge

By now many of the options used below for `kli vc issue` will be familiar to you. Using the at symbol “`@`” in the arguments instructs the command to retrieve the contents of the file indicated by the file path. The command options in addition to `--name` and `--alias` describe above include:
- `--registry-name` as the registry from which to issue the credential.
- `--schema` indicating which ACDC schema to use. Value is the SAID from the “$id” field of the schema you are using to issue the credential with.
- `--recipient` indicates who is receiving the issued ACDC. The value is the AID prefix for the recipient of the ACDC. An OOBI must resolved for the recipient’s prefix so that the issuer controller knows where to send the ACDC.
- `--data` is the credential-specific data that is described by the “attributes” section of schema referenced by the `--schema` argument.
- `--edges` is the JSON object that contains the edge definitions to link the credentials together.
- `--rules` is the JSON object containing the Ricardian Contract definitions for this particular ACDC.


##### Data

An example of the data for this credential is as follows:

```json
{
  "requester": {
    "firstName": "Richard",
    "lastName": "Ayris",
    "nickname": "Dunkie"
  },
  "desiredPartySize": 2,
  "desiredSplit": 50.00
}
```

##### Command

```bash
kli vc issue --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --registry-name ${EXPLORER_REGISTRY} \
  --schema "${JOURNEY_MARK_REQUEST_SCHEMA_SAID}" \
  --recipient "${WISEMAN_PREFIX}" \
  --data @"${ATHENA_DIR}"/credential_data/journey-mark-request-data-richard.json \
  --edges @"${RICHARD_JOURNEY_EDGE}" \
  --rules @"${ATHENA_DIR}"/credential_rules/journey-mark-request-rules.json
```

#### List Credentials to see newly issued Request

Once again we wait for the credential to be issued with `**kli vc list ... --poll**` using the `**--schema**` SAID of the JourneyMarkRequest schema. This polls for all credentials the wise man has received under this schema.

```bash
kli vc list --name ${WISEMAN_KEYSTORE} \
  --alias ${WISEMAN_ALIAS} \
  --schema "${JOURNEY_MARK_REQUEST_SCHEMA_SAID}" \
  --poll
```

This will provide output similar to the following:

```bash
Checking mailboxes for any received credentials......
Current received credentials for ramiel (EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ):
Credential #1: EDq5iBiirGerg2gIE1c95VdJd_SZ7W0S8uq6mP2rG8V8
    Type: Journey Mark Request Credential
    Status: Issued ✔
    Issued by EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q
    Issued on 2023-03-28T22:20:03.452772+00:00
```

### Issue JourneyMarkRequest Credential – Agent (Mark I – deprecated)

Performing the issuance of the JourneyMarkRequest with the Mark I Agent is very similar to using. the KLI. The primary difference is that the JSON files passed in to `--data`, `--edges`, and `--rules` are placed inside the JSON body of the HTTP POST request to the `/credentials/{alias}/` route.

#### Prepare TreasureHuntingJourney Edge

The preparation steps here are pretty much identical to preparing the TreasureHuntingJourney edge for the KLI version of this section.

```bash
# Add SAID to JourneyMarkRequest rules
kli saidify --file "${ATHENA_DIR}"/credential_rules/journey-mark-request-rules.json
RICHARD_JOURNEY_EDGE_FILTER=${ATHENA_DIR}/credential_edges/richard-journey-edge-filter.jq
RICHARD_JOURNEY_EDGE=${ATHENA_DIR}/credential_edges/richard-journey-edge.json
echo "{d: \"\", journey: {n: ., s: \"${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}\"}}" >"${RICHARD_JOURNEY_EDGE_FILTER}"
EXPLORER_JOURNEY_SAID=$(curl -s -X GET "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}?type=received&schema=${TREASURE_HUNTING_JOURNEY_SCHEMA_SAID}" | jq '.[0] | .sad.d' | tr -d '"')
echo \""${EXPLORER_JOURNEY_SAID}"\" | jq -f "${RICHARD_JOURNEY_EDGE_FILTER}" >"${RICHARD_JOURNEY_EDGE}"
kli saidify --file "${RICHARD_JOURNEY_EDGE}"
```

See the “List Credentials to see newly issued request” section below for an explanation of the Bash piping used in the preceding commands.

Once you have the SAIDified edge file ready then you use it to issue the JourneyMarkRequest credential.

#### Issue JourneyMarkRequest Credential

To simplify issuing this credential from the `workflow.sh` script the contents of the JSON files for the data, rules, and edges are stored in BASH variables using the `**VARNAME=$(cat FILENAME)**` pattern. This makes it easy to include the contents in the `curl` command below.

```bash
REQUEST_RULES=$(cat "${ATHENA_DIR}"/credential_rules/journey-mark-request-rules.json)
RICHARD_MARK_DATA=$(cat "${ATHENA_DIR}"/credential_data/journey-mark-request-data-richard.json)
RICHARD_EDGE_DATA=$(cat "${RICHARD_JOURNEY_EDGE}")
curl -s -X POST "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}" \
  -H "accept: application/json" \
  -H "Content-Type: application/json" \
  -d "{\"credentialData\":${RICHARD_MARK_DATA},
       \"recipient\":\"${WISEMAN_PREFIX}\",
       \"registry\":\"${EXPLORER_REGISTRY}\",
       \"schema\":\"${JOURNEY_MARK_REQUEST_SCHEMA_SAID}\",
       \"source\":${RICHARD_EDGE_DATA},
       \"rules\":${REQUEST_RULES}}" \
  | jq '.d' | tr -d '"'
# since there is no polling then sleep to wait for credential delivery
sleep 5
```

There doesn’t exist an equivalent to `kli vc list ... --poll` in the KERIpy Mark I agent so you can perform a `sleep 5` or something similar for an equivalent wait to ensure that a request to `**/credentials/{alias}/type=issued&schema={schema_said}**` will return the issued credential. You could also manually poll this endpoint, process the JSON array of credentials returned, and then continue to poll if the return body doesn’t contain the expected credential.

Next we move on to reviewing the listed credentials with a GET request.

#### List Credentials to see newly issued request

Make a GET request to the `**/credentials/{alias}/type=issued&schema={schema_said}**` endpoint on a Mark I agent to get all credentials issued by that agent’s controller of the specified schema type. Below. I use [command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html) to encapsulate this sort of request as well as a pipe to jq and tr in order to retrieve the credential SAID for later use in the `workflow.sh` script.

```bash
EXPLORER_REQUEST_CRED_SAID=$(curl -s -X GET "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}?type=issued&schema=${JOURNEY_MARK_REQUEST_SCHEMA_SAID}" | jq '.[0] | .sad.d' | tr -d '"')
```

Executing the GET request to a controller, in this case the Wise Man, without the command substitution returns a response that looks similar to the following, with some attributes minimized:

{% image(path="/images/posts/07-abydos-image-6.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

You can see the first array index item “jq ‘.[0]'” and the path “.sad.d” used to get the SAID (digest) of the first credential that is returned from the command. Again, the use of **tr** here is to remove the double quotes to simplify things in the workflow.sh script.

There will be more than one credential returned if you issue this GET request after running the `workflow.sh` script.

### Issuance Wrap Up

#### Issuing the JourneyMark Credential

Issuing the JourneyMark credential is essentially the same process as the JourneyMarkRequest issuance only with different data, edges, and rules. As a challenge for you try to issue this credential without reading the code in the [issue_journeymark_credentials](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L834) and [issue_journeymark_credentials_agent](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L883) functions in `workflow.sh`.

Once the JourneyMark credentials have been issued your issuance state will look like the following:

{% image(path="/images/posts/07-abydos-credentials-jm.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

As you can see the JourneyMark credentials have been issued from Ramiel to both Richard and Elayne.

#### Issuing the JourneyCharter Credential

The presence of two edges in the edge file is the only substantive difference in issuing this credential as compared to the issuance of both the JourneyMarkRequest and the JourneyMark credential issuance processes. The finished edge file looks like this:

```json
{
  "d": "",
  "mark": {
    "n": "EI4Z_hKC7hzWfslkhGD8tGv0eMQCBpmVXx-sJ3Xugz8_",
    "s": "EBEefH4LNQswHSrXanb-3GbjCZK7I_UCL6BdD-zwJ4my"
  },
  "journey": {
    "n": "EPxgzkyYW6VdIx0cazHxioyiNMGDl01jlZwPD71hbBae",
    "s": "EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l"
  }
}
```

As an added challenge to top it all off for this section take your knowledge of schema writing, schema linking, edge creation, and credential issuance and make your best attempt at assembling your own JourneyCharter edges file for use in issuing the JourneyCharter credential. If you get stuck you can look at the code in the [issue_journeycharter_credentials](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#LL937C10-L937C42) and [issue_journeycharter_credentials_agent](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L998) functions from the `workflow.sh` script.

### Final Credential Issuance State

Once all of the credentials have been issued you will have the state of your credential issuance as shown in the below picture.

{% image(path="/images/posts/07-abydos-credentials-all-1.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

This shows that Ramiel has issued the JourneyCharter to both Richard and Elayne.

With all of the credentials issued we can finally move on to one of the most exciting parts, Credential Presentation.

## Credential Presentation

In order for a decentralized identity credentialing system to have any value it must be able to plug into a business process somewhere. Credential presentation is the beginning of plugging in a credentialing system to your custom business process logic. Your primary option here is to write a custom controller. The “[Sally](https://github.com/GLEIF-IT/sally)” project is one such custom controller. It treats KERIpy as a library on top of which to layer in your custom business logic. I did precisely this in [my fork](https://github.com/kentbull/sally) of Sally (a.k.a. Abydos Gatekeeper), which I explain below.

Custom business logic must plug directly in to the credential processing workflow within a controller and respond to credential validation actions that occur when a credential is presented based on whether the credential is issued or revoked. The point at which the custom business logic comes into contact with the decentralized identity system are in custom handler functions written in your custom controller.

A webhook is called from the custom handler functions to illustrate how you would signal controller events to an app, possibly a wallet app, waiting to be notified of event completions from the KERI Agent.

### Presentation Handling Setup

In order to respond to the credential presentation we must do the following setup steps:
1. Ensure version 0.6.0 of the [kentbull/sally](https://github.com/kentbull/sally) fork (a.k.a. Abydos Gatekeeper) is installed with `python -m pip install -e ./` from the fork repo root.
2. Write a credential type mapping file similar to the [schema-mappings.json](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/schema-mappings.json) file that pairs the ACDC schema SAIDs you wrote with credential type names that will be keyed off of in the custom controller. This is important so that the schema SAIDs you generated in your schema linking step match up with the ones the Abydos Gatekeeper (sally) expects.
3. Start the Abydos Gatekeeper (sally) with the appropriate command line arguments including the mapping file you wrote (or copied from the tutorial repo).
4. Start the webhook on the same port you specified when you started the Abydos Gatekeeper
5. Present the credential.
6. Await the webhook’s response to the credential.


#### Examples in workflow.sh script

All of these steps, except dependency installation, are written into the `workflow.sh` script.
- The schema mappings file writing occurs on [line 199 inside read_schema_saids](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L199). Once again jq is used, this time with the `--null-input` argument, to write a JSON file based on a jq filter file, the [schema-mappings-filter.jq](https://github.com/TetraVeda/abydos-tutorial/blob/master/athena/schemas/schema-mappings-filter.jq) file.
- The gatekeeper is started up in the [start_gatekeeper](https://github.com/TetraVeda/abydos-tutorial/blob/master/workflow.sh#L512) function. You will see this includes the `--schema-mappings` argument that expects the schema mappings file you wrote earlier.This also includes the `--web-hook` argument. Set this to be the web hook you started earlier. This will be `http://127.0.0.1:9923`.Finally, the `--auth` argument must be set to the AID prefix of the Wise Man Ramiel so that the schema validation functions can allow Richard’s and Elayne’s JourneyCharter credentials to pass validation.


Next we walk through the code customizations made to the Sally server that enable responding to ATHENA-specific ACDC schema types.

### Abydos Gatekeeper : Custom Controller Code for credential handlers

This is the most complicated, least explained portion of this tutorial. If you want to stay out of the inner details of the custom controller and out of KERIpy then just follow the commands listed in the `workflow.sh` script. On the other hand, if you are willing to venture into this depth I will assume you have some familiarity with Python as well as the bravery to confront the innards of the flow-based programming, hierarchical structured concurrency asynchronous runtime library Dr. Smith wrote, [ioflo/hio](https://github.com/ioflo/hio), that is at the core of both KERIpy and the Sally custom controller.

After reading through the controller code, or KERIpy code, you may find yourself wanting to write a custom controller either with a different async framework or another language, which are reasonable desires. I encourage you to dig a little deeper into the HIO code and the KERIpy code before rewriting it. I expect to have a future blog post on the HIO library as well as its usage in KERIpy, though do what. you think is most reasonable. I offer this suggestion just to encourage you to become familiar with the core KERI internals mostly so we can increase the core contributor count to KERIpy. Many hands make light work.

#### Communicator class

The [Communicator](https://github.com/kentbull/sally/blob/dev/src/sally/core/handling.py#L113) class in `sally/src/sally/core/handling.py` is our main focus. The primary difference between the GLEIF-IT upstream repository and my fork of Sally is that I made a dict of schema handlers keyed by the names of credentials so it would be easy to swap in new schema SAIDs to be recognized as I wrote the schema files during initial development of the schemas. I also thought it readable and human-friendly to use schema type names like “TreasureHuntingJourney” as dict keys rather than schema prefixes like “EIxAox3KEhiQ_yCwXWeriQ3ruPWbgK94NDDkHAZCuP9l”.

```bash
# line 107-110 in handling.py
JOURNEY_TYPE = 'TreasureHuntingJourney'
REQUEST_TYPE = 'JourneyMarkRequest'
MARK_TYPE = 'JourneyMark'
CHARTER_TYPE = 'JourneyCharter'
```

Technically you could hardcode the schema prefixes into your code since once you standardize on a specific set of ACDC schemas and they are in production then you won’t be changing them especially since you must support backwards compatibility likely indefinitely for whatever schemas you release to production. Yet, even with this technical possibility, I prefer to use schema type names so I wrote a a resolver function to resolve a schema.

##### Communicator init function

You can see in the init function I made a dict for schema handlers and another dict for payload handlers that maps the type names above to specific functions.

{% image(path="/images/posts/07-abydos-image-7.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

I then use two resolver functions, type to SAID and SAID to type, to pair up schema SAIDs from incoming presentations to the configured SAIDs from the schema SAID mapping file we wrote earlier:

```python
def resolve_said_to_type(self, schema_said):
    for mapping in self.mappings:
        if mapping.said == schema_said:
            return mapping.credential_type
    raise kering.ValidationError(f"no mapping found for schema {schema_said}")
def resolve_type_to_said(self, credential_type):
    for mapping in self.mappings:
        if mapping.credential_type == credential_type:
            return mapping.said
    raise kering.ValidationError(f"no mapping found for schema {credential_type}")
```

If no mapping is found for a given type then a ValidationError is raised.

##### Schema Validator Function Selection and Invocation

After resolving the schema type for an incoming credential then a handler is selected from the configured SAID mapping file as shown [on line 185](https://github.com/kentbull/sally/blob/dev/src/sally/core/handling.py#L185) in order to select and then invoke a credential validator:

{% image(path="/images/posts/07-abydos-image-9.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

##### Payload Handler Function Selection and Invocation

After selecting the validator, and assuming the credential passes validation, a response payload handler is selected [on line 246](https://github.com/kentbull/sally/blob/dev/src/sally/core/handling.py#L246) and then invoked.

{% image(path="/images/posts/07-abydos-image-8.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

Again, if no schema validator or handler is found an exception is raised. Otherwise the correct handler is selected to validate the schema and then the correct payload handler prepares the response to the presentation event.

Both of the following functions are the most complicated versions of their functions in Abydos Gatekeeper. You can read through the other similar functions if you want more illustrations.

##### JourneyCharter schema validator

As you can see in the [validateJourneyCharter](https://github.com/kentbull/sally/blob/dev/src/sally/core/handling.py#LL414C12-L414C12) function first the ACDC schema SAID is validated, the credential issuer is verified, and then the credential is handed to other validation functions to validate the full credential chain. Refer to the code for a complete understanding of what it means to validate the full credential chain.

{% image(path="/images/posts/07-abydos-image-10.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

##### JourneyCharter payload handler

In response to a credential presentation the custom controller can return a customized object which. in this case is the data dict beginning at line 511. What makes this payload function more interesting is the use of the registry (reger) object to retrieve data from each of the credentials chained to the JourneyCharter credential.

{% image(path="/images/posts/07-abydos-image-11.png", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

With all of this setup complete we are ready to set up the webhook.

### Webhook Setup

The webhook only starts up a HTTP server on the default port of 9923. You start it up with the following:

```bash
$ sally hook demo
# Should provide output like the following:
launching on port 9923
Sally Web Hook Sample listening on 9923
```

Once the webhook is listening then you are ready to make a presentation and receive a response from the controller.

### Present JourneyCharter Credential – KLI

Making a presentation request requires knowing the SAID of the credential you want to present as well as the alias of the recipient you want to send the presentation to. This alias must match up the OOBI alias name you used earlier when resolving an OOBI to connect the presenting controller to the target controller the presentation is being sent to.

```bash
# First we get the SAID of the JourneyCharter credential for Richard
RICHARD_CRED_SAID=$(kli vc list --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --said \
  --schema ${JOURNEY_CHARTER_SCHEMA_SAID})
# Then we present the credential to the Gatekeeper
kli vc present --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --said "${RICHARD_CRED_SAID}" \
  --recipient ${GATEKEEPER_ALIAS} \
  --include
```

The `--include` option instructs the KLI to send all of the ACDC data in the presentation and is critical in order for the request to work.

Once the presentation is sent you will need to wait a few moments for the target controller to receive the credential, validate it, and then send the response payload to the webhook.

#### Webhook Response

All the webhook does is return a dump of the HTTP headers and the HTTP request body. For example, the Gatekeeper’s response to Richard’s presentation of the JourneyCharter credential looks like the following, and Elayne’s is very similar:

```bash
Gatekeeper | received request
Gatekeeper | Valid Credential. Validated at 2023-03-28 19:02:52.536642
*** HEADERS **
{
  "CONTENT-TYPE": "application/json",
  "CONTENT-LENGTH": "705",
  "HOST": "127.0.0.1:9923",
  "ACCEPT-ENCODING": "identity",
  "CONNECTION": "close",
  "SALLY-RESOURCE": "EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr",
  "SALLY-TIMESTAMP": "2023-03-29T01:02:52.473468+00:00",
  "SIGNATURE-INPUT": "sig0=(\"sally-resource\" \"@method\" \"@path\" \"sally-timestamp\");created=1680051772;keyid=\"TPFGOhkQykkgWHugX1csJrZQSmOfFqEbxueLSWUMZZU=\";alg=\"ed25519\"",
  "SIGNATURE": "indexed=\"?1\";signer=\"EMHY2SRWuqcqlKv2tNQ9nBXyZYqhJ-qrDX70faMcGujF\";sig0=\"51ajn3xM8d8QTvEM61msYyFjrXEmC1VXkUi73wAve0md1LUyRIfgYmdb0G-DRH3-DLzADp_F77gq04N_0TbdBQ==\""
}
**************
**** BODY ****
{
  "action": "iss",
  "actor": "EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ",
  "data": {
    "schema": "EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr",
    "issuer": "EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ",
    "issueTimestamp": "2023-03-29T01:02:02.818290+00:00",
    "credential": "EG9XT6BF5yzPPbJIfO_iSZVYn0il0l99wfMHmbxvVSz_",
    "recipient": "EJS0-vv_OPAQCdJLmkd5dT0EW-mOfhn_Cje4yzRjTv8q",
    "partySize": 2,
    "authorizerName": "Ramiel",
    "journeyCredential": "EMg4Za1dqn2pwoHJdR4NL5sgSC3z9yk650DyT_wxJUKj",
    "markCredential": "EMgj_5tQJejZXrTO_su2fblS8MEU-avRJb6yqsJMQS-n",
    "destination": "Osireion",
    "treasureSplit": "50/50",
    "journeyEndorser": "Ramiel",
    "firstName": "Richard",
    "lastName": "Ayris",
    "nickname": "Dunkie"
  }
}
**************
```

### Present JourneyCharter Credential – Agent (Mark I – deprecated)

Presenting with the agent is very similar to with the KLI. You must know the recipient alias, corresponding to the “oobialias” property used to register an OOBI earlier, you must know the ACDC schema SAID, as well as the SAID of the credential you want to present.

```bash
EXPLORER_CHARTER_CRED_SAID=$(curl -s \
  -X GET "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}?type=received&schema=${JOURNEY_CHARTER_SCHEMA_SAID}" \
  | jq '.[0] | .sad.d' | tr -d '"')
curl -s \
  -X POST "${EXPLORER_AGENT_URL}/credentials/${EXPLORER_ALIAS}/presentations" \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d "{
        \"recipient\": \"${GATEKEEPER_ALIAS}\",
        \"said\": \"${EXPLORER_CHARTER_CRED_SAID}\",
        \"schema\": \"${JOURNEY_CHARTER_SCHEMA_SAID}\",
        \"include\": true
      }"
```

Once again the `**"include": true**` property is essential to instruct the agent to send all of the ACDC data for the JourneyMark credential and all linked credentials to the recipient controller, the Abydos Gatekeeper.

## Credential Revocation

Revocation is an essential part of the lifecycle of an ACDC-style credential. A credential may be revoked at any time. This is represented as a “rev” action in the TEL for that ACDC and can be included in either an “ixn” (Interaction) or “rot” (Rotation) event in the controller’s KEL.

### Revoke JourneyCharter Credential – KLI

Revoking a credential is very simple. Starting with the SAID of the credential to be revoked the below example stores it in the BASH variable `RICHARD_JOURNEY_CHARTER_CRED_SAID` which is then fed into the `kli vc revoke` command.

```bash
RICHARD_JOURNEY_CHARTER_CRED_SAID=$(kli vc list \
  --name ${EXPLORER_KEYSTORE} \
  --alias ${EXPLORER_ALIAS} \
  --said \
  --schema ${JOURNEY_CHARTER_SCHEMA_SAID})
kli vc revoke --name ${WISEMAN_KEYSTORE} \
  --alias ${WISEMAN_ALIAS} \
  --registry-name ${WISEMAN_REGISTRY} \
  --said "${RICHARD_JOURNEY_CHARTER_CRED_SAID}" \
  --send ${GATEKEEPER_ALIAS}
```

Then when this credential is presented again later the Gatekeeper will see that the credential has been revoked.

The `--send` argument is essential to instruct the issuing controller to send the revocation notice to the target controller, in this case the Gatekeeper.

#### Webhook revocation notification

With the revocation information the Gatekeeper adjusts its internal state for that credential and then triggers the webhook with a revocation payload. The output from the webhook looks like the following:

```bash
Gatekeeper | received request
Gatekeeper | Invalid credential. Revoked on: 2023-03-29T15:17:19.394845+00:00
*** HEADERS **
{
  "CONTENT-TYPE": "application/json",
  "CONTENT-LENGTH": "263",
  "HOST": "127.0.0.1:9923",
  "ACCEPT-ENCODING": "identity",
  "CONNECTION": "close",
  "SALLY-RESOURCE": "EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr",
  "SALLY-TIMESTAMP": "2023-03-29T15:17:32.041445+00:00",
  "SIGNATURE-INPUT": "sig0=(\"sally-resource\" \"@method\" \"@path\" \"sally-timestamp\");created=1680103052;keyid=\"TPFGOhkQykkgWHugX1csJrZQSmOfFqEbxueLSWUMZZU=\";alg=\"ed25519\"",
  "SIGNATURE": "indexed=\"?1\";signer=\"EMHY2SRWuqcqlKv2tNQ9nBXyZYqhJ-qrDX70faMcGujF\";sig0=\"dTZ6HvZTrQrPVmrDiGUs7pv9xuUJyjFmGFZdsda5O3WSloip5LnCg5cvE8hr7jTEIFNN1_H8Dwx-s-Yc2fEoCg==\""
}
**************
**** BODY ****
{
  "action": "rev",
  "actor": "EIaJ5gpHSL9nl1XIWDkfMth1uxbD-AfLkqdiZL6S7HkZ",
  "data": {
    "schema": "EEq0AkHV-i5-aCc1JMBGsd7G85HlBzI3BfyuS5lHOGjr",
    "credential": "EDRCzS5tb0BNuaWOZ5dGmlcBHPZNtWvT1zrcZBh8icTP",
    "revocationTimestamp": "2023-03-29T15:17:19.394845+00:00"
  }
}
**************
```

### Presenting a Revoked Credential

When you try to present the credential again to the Gatekeeper the revocation status is detected.

```bash
kli vc present --name explorer --alias richard \
  --said ${RICHARD_JOURNEY_CHARTER_CRED_SAID} \
  --recipient zaqiel \
  --include
```

Yet the Gatekeeper detects the revoked status:

```bash
revoked credential EIdNBtudTLx3g-WWIYmuvzkUmVnXgT3EdaUC-3JbeAv1 being presented
# ...continues on to send revoked credential to the webhook
```

The one thing missing from the Abydos Gatekeeper for this revocation flow is to send a different payload to the webhook when a revoked credential is presented. This topic will be explored in a future blog post.

With that complete we move on to the bonus section on revocation with the Mark I agent.

### Bonus – Revoke JourneyCharter Credential – Agent (Mark I – deprecated)

A simple revocation is possible with the Mark I Agent yet it is missing functionality similar to the `--send` argument to `kli vc revoke` and thus should not be used. It is included here as an illustration of how revocation with an agent may work.

```bash
curl -s \
  -X DELETE "${WISEMAN_AGENT}/credentials/${WISEMAN_ALIAS}?registry=${WISEMAN_REGISTRY}&said=${EXPLORER_JOURNEY_CHARTER_CRED_SAID}" \
  --header 'accept: */*' \
  --header 'Content-Type: application/json'
```

The response is an HTTP 202 Accepted and a blank response body.

## Congratulate yourself!

Your new network is now ready to send our brave treasure hunters on their journey to the Osireion at Abydos! They, and ATHENA, will be very pleased with you and your work. Maybe you will even get a cut of the treasure.

{% image(path="/images/posts/07-abydos-treasure-hunting.jpg", class="diagram",
    op="fit_width",
    alt="") 
    %}
Image
{% end %}

Now you know how to set up an entire KERI network from scratch including:
1. Writing ACDC schemas
2. Linking ACDC schema graphs
3. Writing all deployment configuration files for witnesses and controllers
4. Setting up witnesses
5. Setting up and initializing keystores
6. Performing inception events
7. Setting up agents (even if the Mark I deprecated agents)
8. Performing OOBI introductions
9. Issuing ACDCs
10. Chaining ACDCS and issuing chained ACDCs
11. Writing a custom controller with custom schema validators and response payload constructors
12. Setting up a webhook to listen to credential lifecycle events
13. Presenting a chained ACDC to a custom controller
14. Observing presentation events with a custom controller and webhook
15. Revoking an ACDC
16. Presenting a revoked credential to a custom controller
17. Observing the revocation event from a custom controller and webhook


Congratulate yourself for all your hard work and revel in your newfound understanding of the KERI and ACDC decentralized identity stack!

## Missing Parts – Future Additions

Even though this blog post is long there is plenty we did not cover that I would like to cover on future posts. This includes the following, as well as the list at the start of the post in [Concepts Not Covered](#concepts-not-covered).

Mobile app and Edge Signing

Watcher networks

Mark II Agent – KERIA

CESR – CESRide, Parside, Signify-TS

## The End!

See you in the next article

## Acknowledgments

Philip Feairheller – for the awesome code in Sally that was the bulk of the code used for the Abydos Gatekeeper, for your work in KERIpy, and everywhere in the KERI and ACDC space.

Kevin Griffin – for the original code in `generate.py` from the vLEI-server that was the seed for creating KASLCred, as well as your many other contributions to KERIpy and the other repos in the KERI and ACDC space.

And of course, Dr. Samuel M. Smith for your awesome, elegant work with KERI and ACDC.

Nuttawut Kongsuwan for your definitions document “KERI jargon in a nutshell (Part 1)”

Jim Martin for your windows installation instructions.

Henk Van Cann for maintaining the ACDC terms wiki, a useful reference while building this post.

Steven Milstein and the Switchchord team for initial reviews and first looks at the article.

## References

[Authentic Chained Data Containers IETF Draft Specification](https://trustoverip.github.io/tswg-acdc-specification/draft-ssmith-acdc.html) – Sam Smith

[Presentations folder](https://github.com/SmithSamuelM/Papers/tree/master/presentations) of Sam Smith’s Papers Github Repository

[KERI Agents documentation](https://hackmd.io/ztr6psSETIWiRciWDHpt2Q?view) (HackMD doc) – Rodolfo the KERI Cardano Wizard (RKCW)

[KERI jargon in a nutshell](https://medium.com/finema/keri-jargon-in-a-nutshell-part-1-fb554d58f9d0) (36 jargon explanation snippets) – Nuttawut Kongsuwan

[ACDC and KERI Terms Wiki](https://github.com/trustoverip/acdc) (385 terms at present) – Henk Van Cann et. al.

[Lightning Memory Mapped Database](http://www.lmdb.tech/doc/) – Symas Corp.

## Image Credits

Abydos Temple: [https://www.thenotsoinnocentsabroad.com/blog/the-serene-spirituality-of-abydos-temple](https://www.thenotsoinnocentsabroad.com/blog/the-serene-spirituality-of-abydos-temple)

If I missed you let me know and I will ad you here.

Many pictures were Creative Commons licensed.

