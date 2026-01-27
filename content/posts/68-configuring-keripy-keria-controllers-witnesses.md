+++
title = "Configuring KERIpy Controllers, Witnesses, and KERIA Agents"
slug = "configuring-keripy-keria-controllers-witnesses"
date = "2026-01-26"

[taxonomies]
tags=["keri", "keria", "keripy", "configuration", "witnesses", "controllers", "agents", "iurls", "curls", "durls", "oobi"]

[extra]
comment = true
+++

# Configuring KERIpy Components, KERIA Agents, and Well-known URIs

How do you configure, publish, discover, and connect KERI components? Whether witnesses, verifiers, KERIA deployments, mailboxes, or general KERI controllers, configuration touches the topics of node publication, discovery, and connection, data discovery, and service endpoints. Node discovery and connection occurs with out of band identifiers (OOBIs). Data discovery of ACDC schemas and contents also utilize with OOBIs. Service endpoint configurations combine location scheme records and endpoint role authorizations. This article explains these concepts, KERI configuration mechanisms, and how to use them for real world, production configurations.

**Disclaimer**: This article assumes some basic familiarity with KERI concepts including identifiers (AIDs), witnesses, OOBIs, and the difference between KERIpy (the KERI implementation) and KERIA (the agent service). You should be able to follow along if you have some experience with KERIpy, the KLI, or KERIA and SignifyTS. If you are new to KERI and KERIA then go to the [vLEI Trainings](https://github.com/GLEIF-IT/vlei-trainings) repository and work through those trainings to have a proper introduction and then return to this configuration guide.

## Introduction: The Configuration Challenge

Configuring KERIpy controllers and KERIA agents occurs at multiple levels:

**Node Configuration**
1. **Controller Configuration** (KERIpy) - How your local identifier finds witnesses and peers, and which ports it listens on, if any.
  - This is also used as the basis for **verifier configuration** in components like [sally](https://github.com/GLEIF-IT/sally) or the [vLEI Verifier](https://github.com/GLEIF-IT/vlei-verifier).
2. **Witness Configuration** (KERIpy) - The endpoints witnesses listen on
3. **Agency Configuration** (KERIA) - Global settings for the KERIA service
4. **Agent Configuration** (KERIA) - Per-agent settings within KERIA
5. **Mailbox Configuration** (KERIpy) - Where receipts and messages are delivered to

**Content Hosting**
6. **ACDC Schema Hosting** (KERIpy) - Hosting ACDC schemas for resolution by the OOBI URL resolution process.
7. **ACDC Content Hosting** (KERIpy) - Hosting ACDC content for resolution by the OOBI URL resolution process.

**Well Known Discovery Configuration**
8. **Well-known URIs for Hosted Components** - `/.well-known/keri/oobi/` general discovery endpoints for KERI OOBIs, ACDC schemas and witness infrastructure

Let's explore each, starting with file-based configuration and progressing to runtime endpoint role authorization.

## Part 1: KERIpy Controller Configuration

Controller configuration applies to components like verifiers, such as the [sally][SALLY_REPO] verifier service that runs its own direct-mode listeners, or the KERIA service that listens on three ports.

### The Configer Class

At the heart of KERIpy configuration is the `Configer` class in `keri/app/configing.py`. It's a specialized file handler supporting four serialization formats:

```python
class Configer(filing.Filer):
    """
    Habitat Config File
    
    Supports four serializations: HJSON, JSON, MGPK (MsgPack), and CBOR
    The serialization is determined by the file extension (.json, .mgpk, .cbor)
    """
    TailDirPath = "keri/cf"
    
    def __init__(self, name="conf", base="main", filed=True, 
                 fext="json", human=True, **kwa):
        # Creates file at path like: ~/.keri/cf/main/conf.json
```

Configuration files follow this structure:

```json
{
  "dt": "2022-01-01T00:00:00.000000+00:00",
  "<AID name or prefix>": {
    "dt": "2022-01-20T12:57:59.823350+00:00",
    "curls": ["tcp://127.0.0.1:12344/", "http://127.0.0.1:12345/"]
  },
  "iurls": [
    "http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha/controller"
  ],
  "durls": [
    "http://127.0.0.1:7723/oobi/EMhvwOlyEJ9kN4PrwCpr9Jsv7TxPhiYveZ0oP3lJzdEi"
  ],
  "wurls": [
    "http://127.0.0.1:5644/.well-known/keri/oobi/EBNa..."
  ]
}
```


### Example: Sally verifier - configuring listening endpoints

The "sally" property of the JSON object corresponds to the `--alias` argument to `kli incept`. The `curls` section of the `sally` object tells the verifier which port to listen on.

Located at your `config_dir/keri/cf/sally.json` you would find the following JSON. And the `sally.json` at the end of the path is really the `<alias>.json` where `alias` corresponds to the `--alias` argument to `kli incept`.
```json
{
  "dt": "2022-10-31T12:59:57.823350+00:00",
   "sally": {
    "dt": "2022-01-20T12:57:59.823350+00:00",
    "curls": ["http://127.0.0.1:9723/"]
  },
  "iurls": [],
  "durls": [
    "http://127.0.0.1:7723/oobi/EBNaNu-M9P5cgrnfl2Fvymy4E_jvxxyjb70PRtiANlJy",
    "http://127.0.0.1:7723/oobi/EH6ekLjSr8V32WyFbGe1zXjTzFs9PkTYmupJ9H65O14g",
    "http://127.0.0.1:7723/oobi/EKA57bKBKxr_kN7iN5i7lMUxpMG-s19dRcmov1iDxz-E",
    "http://127.0.0.1:7723/oobi/ENPXp1vQzRF6JwIuS-mp2U8Uf1MoADoP_GqQ62VsDZWY",
    "http://127.0.0.1:7723/oobi/EBfdlu8R27Fbx-ehrqwImnK-8Cm79sqbAQ4MmvEAYqao",
    "http://127.0.0.1:7723/oobi/EEy9PkikFcANV1l7EHukCeXqrzT1hNZjGlUk7wuMO5jw"
  ]
}
```

The `iurls` section is for witness OOBIs. The `durls` section is for data OOBIs, usually ACDC schemas or ACDC components. This example config does not have `wurls` for well knowns.

### Example: KERIA agent server - configuring listening endpoints

Similar to the Sally verifier, the `keria` property in the config corresponds to the value passed to the `--name` argument to `keria start` which defaults to `keria`.

Again, you see the `curls` section for configuring the HTTP port used for listening to CESR transmissions to an agent. It has `iurls` and then KERIA-specific configuration for async loop repeat times, called "tocks."

```json
{
  "dt": "2025-01-03T16:08:30.123456+00:00",
  "keria": {
    "dt": "2025-01-03T16:08:30.123457+00:00",
    "curls": ["http://127.0.0.1:3902/"]
  },
  "iurls": [
    "http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha/controller?name=Wan&tag=witness",
    "http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM/controller?name=Wil&tag=witness",
    "http://127.0.0.1:5644/oobi/BIKKuvBwpmDVA4Ds-EpL5bt9OqPzWPja2LigFYZN2YfX/controller?name=Wes&tag=witness"
  ],
  "tocks": {
    "initer": 0.0,
    "escrower": 1.0
  }
}
```

**Configuration URL Types:**

- **`curls`** (Controller URLs): URLs + ports on which to set up either a TCP or HTTP port listener - where this controller can be reached
- **`iurls`** (Introduction URLs): OOBI URLs for witnesses, watchers, mailboxes, and any KERI controller to resolve at startup
- **`durls`** (Data URLs): OOBI URLs for static data like ACDC schemas or credentials
- **`wurls`** (Well Known URLs): OOBI URLs specifically for well known components


## Part 2: Witness Configuration

Witnesses are non-transferable identifiers that provide receipt services. Their configuration differs from controllers:

### Witness Endpoint Configuration

Witnesses need two types of configuration:

1. **Controller endpoints** - Where the witness accepts events
2. **Location schemes** - The URLs where the witness is reachable

In this case the `wan` configuration property corresponds to the `--alias` property passed to the `kli incept` or `kli witness start` command. A witness listens on both a TCP and an HTTP port, as shown in the `curls` section below. 

```json
{
  "dt": "2022-01-20T12:57:59.823350+00:00",
  "wan": {
    "dt": "2022-01-20T12:57:59.823350+00:00",
    "curls": ["tcp://127.0.0.1:5632/", "http://127.0.0.1:5642/"]
  },
  "iurls": [
    "http://127.0.0.1:5643/oobi/BLskRTInXnMxWaGqcpSyMgo0nYbalW99cGZESrz3zapM/controller?name=Wil&tag=witness"
  ],
  "durls": [
    "http://127.0.0.1:7723/oobi/EBNaNu-M9P5cgrnfl2Fvymy4E_jvxxyjb70PRtiANlJy",
  ],
  "wurls": [
    "http://127.0.0.1:7723/.well-known/keri/oobi/EPflJSbTCs2WKoGx4zIJ5OpOXHXuY0JE9et9ile2gMpv"
  ]
}
```

### Witness Discovery

Controllers discover witnesses through:

1. **Configuration `iurls`**: Pre-configured witness OOBIs
2. **OOBI resolution**: Resolving witness OOBIs at runtime
3. **Well-known endpoints**: Using `.well-known/keri/oobi/` discovery

The witnesses must have their endpoint role authorizations configured so the controller can find them.

## Part 3: Mailbox Configuration

Mailboxes are where witnesses and other agents deliver messages. Configuration involves:

### Agent Endpoint Roles

An agent acts as a mailbox by having the `mailbox` endpoint role authorized:

```python
# Configure agent with multiple roles
msgs = bytearray()

# Controller role
msgs.extend(agentHab.makeEndRole(
    eid=agentHab.pre,
    role=kering.Roles.controller,
    stamp=helping.nowIso8601()
))

# Location
msgs.extend(agentHab.makeLocScheme(
    url='http://127.0.0.1:6666',
    scheme=kering.Schemes.http,
    stamp=helping.nowIso8601()
))

# Agent role (for controller to reach agent)
msgs.extend(hab.makeEndRole(
    eid=agentHab.pre,
    role=kering.Roles.agent,
    stamp=helping.nowIso8601()
))

# Mailbox role (for witnesses to deliver messages)
msgs.extend(hab.makeEndRole(
    eid=agentHab.pre,
    role=kering.Roles.mailbox,
    stamp=helping.nowIso8601()
))

agentHab.psr.parse(ims=bytearray(msgs))
hab.psr.parse(ims=bytearray(msgs))  # Controller processes too
```

### Querying Endpoints

Controllers can query their configured endpoints:

```python
ends = hab.endsFor(hab.pre)
# Returns:
# {
#     'agent': {
#         'EBErgFZ...': {'http': 'http://127.0.0.1:6666'}
#     },
#     'controller': {
#         'EGadHcy...': {'http': 'http://127.0.0.1:7777'}
#     },
#     'mailbox': {
#         'EBErgFZ...': {'http': 'http://127.0.0.1:6666'}
#     },
#     'witness': {
#         'BN8t3n1...': {'http': 'http://127.0.0.1:8888'}
#     }
# }
```

## Part 4: KERIA Agency Configuration

KERIA introduces a two-level configuration model: agency-level and agent-level.

### Agency Configuration: KERIAServerConfig

The agency configuration is defined in `keria/app/agenting.py`:

```python
@dataclass
class KERIAServerConfig:
    """Agency-wide configuration"""
    
    # HTTP ports
    adminPort: int = 3901
    httpPort: int | None = 3902
    bootPort: int = 3903
    
    # Master controller
    name: str = "keria"
    base: str = ""
    bran: str = None
    configFile: str = "keria"
    configDir: str = None
    
    # TLS
    keyPath: str = None
    certPath: str = None
    caFilePath: str = None
    
    # Logging
    logLevel: str = "CRITICAL"
    logFile: str = None
    logRequests: bool = False
    
    # Agency settings
    cors: bool = True
    releaseTimeout: int = 86400
    
    # Global OOBIs applied to all agents
    curls: List[str] = field(default_factory=list)
    iurls: List[str] = field(default_factory=list)
    durls: List[str] = field(default_factory=list)
```

**Key Agency Configuration Fields:**

- **Ports**: Three HTTP servers (admin, boot, main)
- **Master controller**: Agency's own identifier credentials
- **Global OOBIs**: Applied to every agent created
  - `curls`: Controller service endpoint OOBIs
  - `iurls`: Introduction OOBIs for witnesses/watchers/mailboxes
  - `durls`: Data OOBIs for schemas/credentials

### Creating an Agency

```python
config = KERIAServerConfig(
    name="my-agency",
    bran="MySecurePasscodeHere",
    adminPort=3901,
    httpPort=3902,
    bootPort=3903,
    curls=["http://localhost:7723/oobi/controller"],
    iurls=["http://localhost:5644/oobi/witness1"],
    durls=["http://schemas.example.com/oobi/schema1"],
    temp=False
)

agency = Agency(
    name=config.name,
    bran=config.bran,
    base=config.base,
    releaseTimeout=config.releaseTimeout,
    curls=config.curls,
    iurls=config.iurls,
    durls=config.durls,
    temp=False
)
```

### Agency Configuration File

Agencies can also load from a config file:

```python
cf = configing.Configer(
    name="keria",
    base="",
    headDirPath="/etc/keria",
    temp=False,
    reopen=True
)

agency = Agency(
    name="keria",
    bran=bran,
    cf=cf,
    temp=False
)
```

AS shown earlier, the config file follows the same structure as KERIpy controller configs:

```json
{
  "dt": "2026-01-21T00:00:00.000000+00:00",
  "keria": {
    "dt": "2026-01-21T00:00:00.000000+00:00",
    "curls": ["http://localhost:7723/oobi/agency-controller"]
  },
  "iurls": [
    "http://localhost:5644/oobi/witness1",
    "http://localhost:5644/oobi/watcher1"
  ],
  "durls": [
    "http://schemas.example.com/oobi/vlei-schema"
  ]
}
```

## Part 5: KERIA Agent Configuration

Each agent in KERIA gets its own configuration derived from the agency configuration.

### Agent Configuration Loading

When creating an agent, the agency loads configuration:

```python
def _loadConfigForAgent(self, caid):
    """
    Loads configuration for an agent by:
    1. Reading agency configuration
    2. Extracting agency subsection
    3. Renaming it to agent-specific name
    4. Merging with environment variable OOBIs
    """
    timestamp = nowIso8601()
    config = dict(self.cf.get() if self.cf is not None else {"dt": timestamp})
    
    # Rename agency section to agent section
    habName = f"agent-{caid}"
    config_name = self.name if self.name else "keria"
    if config_name in config:
        config[habName] = config[config_name]
        del config[config_name]
    else:
        config[habName] = {}
    
    # Set up curls for agent (controller-specific)
    config[habName]["curls"] = config[habName].get("curls", [])
    
    # Set up iurls/durls (shared across all agents)
    config["iurls"] = config.get("iurls", [])
    config["durls"] = config.get("durls", [])
    
    # Merge with environment variables
    if self.curls is not None and isinstance(self.curls, list):
        config[habName]["curls"] = config[habName]["curls"] + self.curls
    
    if self.iurls is not None and isinstance(self.iurls, list):
        config["iurls"] = config["iurls"] + self.iurls
    
    if self.durls is not None and isinstance(self.durls, list):
        config["durls"] = config["durls"] + self.durls
    
    return config
```

**Important Distinction:**

- **`config[habName]["curls"]`**: Controller-specific endpoints (unique per agent)
- **`config["iurls"]`**: Shared introduction OOBIs (witnesses, watchers, mailboxes)
- **`config["durls"]`**: Shared data OOBIs (schemas, credentials)

### Agent Configuration Writing

The agency writes agent-specific config files:

```python
def _writeAgentConfig(self, caid):
    """
    Writes agent configuration as modified copy of agency config
    """
    config = self._loadConfigForAgent(caid)
    cf = configing.Configer(
        name=f"{caid}",
        base="",
        human=False,
        temp=self.temp,
        reopen=True,
        clear=False
    )
    cf.put(config)
    return cf
```

This creates a config file at `~/.keri/cf/{caid}.json` containing:

```json
{
  "dt": "2026-01-21T00:00:00.000000+00:00",
  "agent-EIaGMMWJ...": {
    "dt": "2026-01-21T00:00:00.000000+00:00",
    "curls": [
      "http://localhost:7723/oobi/agency-controller",
      "http://localhost:8080/oobi/my-controller"
    ]
  },
  "iurls": [
    "http://localhost:5644/oobi/witness1"
  ],
  "durls": [
    "http://schemas.example.com/oobi/schema1"
  ]
}
```

## Part 6: Endpoint Role Authorization at Runtime

Beyond file-based configuration, KERI uses endpoint role authorization messages for dynamic configuration.

### Endpoint Role Authorization Messages

These are KERI events that authorize specific endpoints for specific roles:

```python
# Controller endpoint
endRoleMsg = hab.makeEndRole(
    eid=hab.pre,                    # Endpoint identifier
    role=kering.Roles.controller,   # Role type
    stamp=helping.nowIso8601()      # Timestamp
)

# Location scheme
locSchemeMsg = hab.makeLocScheme(
    url='http://127.0.0.1:7777',    # Endpoint URL
    scheme=kering.Schemes.http,     # URL scheme
    stamp=helping.nowIso8601()      # Timestamp
)
```

**Available Roles:**

```python
Roles = Rolage(
    controller='controller', 
    witness='witness', 
    registrar='registrar',
    watcher='watcher', 
    judge='judge', 
    juror='juror', 
    peer='peer', 
    mailbox="mailbox", 
    agent="agent")
```

**Available Schemes:**

```python
Schemes = Schemage(
    tcp='tcp', 
    http='http', 
    https='https')
```

### Processing Endpoint Messages

These messages are parsed into the database:

```python
msgs = bytearray()
msgs.extend(hab.makeEndRole(eid=hab.pre, role="controller", 
                           stamp=helping.nowIso8601()))
msgs.extend(hab.makeLocScheme(url='http://127.0.0.1:7777',
                              scheme=kering.Schemes.http,
                              stamp=helping.nowIso8601()))
hab.psr.parse(ims=msgs)

# Verify stored in database
ender = hab.db.ends.get(keys=(hab.pre, "controller", hab.pre))
assert ender.allowed  # Role is authorized

locer = hab.db.locs.get(keys=(hab.pre, kering.Schemes.http))
assert locer.url == 'http://127.0.0.1:7777'  # Location stored
```

### OOBI Resolution vs File Configuration

The key distinction:

**File Configuration (`curls`, `iurls`, `durls`):**
- Static configuration loaded at startup
- Processed during `Habery.makeHab()` or `reconfigure()`
- Stored in config file permanently

**OOBI Resolution:**
- Dynamic configuration at runtime
- Processed via `hab.db.oobis.pin()` and OOBI resolution
- Results in same endpoint role authorizations

Example:

```python
# File-based
conf = {"curls": ["http://localhost:7723/oobi/controller"]}
cf.put(conf)
hab = hby.makeHab(name="alice")  # Processes curls

# Equivalent OOBI resolution
oobi = "http://localhost:7723/oobi/controller"
obr = OobiRecord(date=helping.toIso8601(helping.nowUTC()))
hab.db.oobis.pin(keys=(oobi,), val=obr)
# ... OOBI resolver processes and creates endpoint records
```

Both methods result in the same database records:
- `hab.db.ends`: Endpoint role authorizations
- `hab.db.locs`: Location schemes

## Part 7: Answering the Original Questions

Now we can answer the Discord questions:

### Q: "How to configure a controller/key store with iurls/curls like with kli init --file?"

**Answer**: Three methods exist:

**Method 1**: File-based initialization

```python
# Create config file
cf = configing.Configer(name="myconfig", temp=False)
conf = {
    "dt": help.nowIso8601(),
    "mycontroller": {
        "curls": ["http://localhost:7723/oobi/controller"]
    },
    "iurls": ["http://localhost:5644/oobi/witness1"]
}
cf.put(conf)

# Initialize Habery with config
hby = habbing.Habery(name="myconfig", cf=cf, temp=False)
hab = hby.makeHab(name="mycontroller")
```

**Method 2**: Programmatic configuration

```python
# Create Habery
hby = habbing.Habery(name="test", temp=True)

# Create config
conf = {
    "dt": help.nowIso8601(),
    "alice": {
        "curls": ["http://localhost:7723/oobi/controller"]
    },
    "iurls": ["http://localhost:5644/oobi/witness1"]
}
hby.cf.put(conf)

# Create hab (processes config)
hab = hby.makeHab(name="alice")
```

**Method 3**: OOBI resolution

```python
# Create hab
hab = hby.makeHab(name="alice")

# Resolve OOBIs at runtime
oobi = "http://localhost:5644/oobi/witness1"
obr = OobiRecord(date=helping.toIso8601(helping.nowUTC()))
hab.db.oobis.pin(keys=(oobi,), val=obr)
# OOBI resolver processes in background
```

### Q: "Is there a good example of running the controller server?"

**Answer**: KERIA provides the production pattern:

```python
from keria.app.agenting import (
    KERIAServerConfig, createAgency, setupDoers, agencyDoist
)

# Configure agency
config = KERIAServerConfig(
    name="keria",
    bran="MySecurePasscodeHere",
    adminPort=3901,
    httpPort=3902,
    bootPort=3903,
    iurls=["http://localhost:5644/oobi/witness1"],
    temp=False
)

# Create agency
agency = createAgency(config, temp=False)

# Setup HTTP servers and doers
doers = setupDoers(agency, config)

# Run with Doist
doist = agencyDoist(doers)
doist.do()
```

For simpler use cases without HTTP servers:

```python
# Just create the agency
agency = Agency(
    name="my-agency",
    bran=bran,
    base="",
    iurls=["http://localhost:5644/oobi/witness1"],
    temp=False
)

# Create agents
agent = agency.create(caid="EIaGMMWJ...")

# Run agency
doist = doing.Doist(limit=0.0, tock=0.03125, real=True)
doist.do(doers=[agency])
```

### Q: "No way for the controller to provide durls/iurls except resolving OOBIs?"

**Answer**: This is partially correct but requires clarification:

**In KERIpy**: Controllers can provide `curls`, `iurls`, and `durls` through:
1. Configuration file (shown above)
2. OOBI resolution at runtime
3. Endpoint role authorization messages

**In KERIA**: The agency configuration provides global `iurls` and `durls` that are inherited by all agents. However:

- **Limitation**: Currently no API endpoint to add agent-specific `curls`/`iurls`/`durls` after agent creation
- **Workaround**: Resolve OOBIs through the `/oobis` endpoint:

```bash
POST /identifiers/{alias}/oobis
{
  "oobialias": "my-witness",
  "url": "http://localhost:5644/oobi/witness1"
}
```

This triggers OOBI resolution which creates the same endpoint records as file-based configuration.

**Improvement Opportunity**: As Daniel noted, adding an API to modify agent configuration would be valuable:

```bash
# Proposed API (not yet implemented)
PUT /config
{
  "iurls": ["http://localhost:5644/oobi/new-witness"],
  "durls": ["http://schemas.example.com/oobi/new-schema"]
}
```

## Part 8: Well-Known OOBI Discovery

Well-known URIs provide standardized discovery endpoints for KERI resources following [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html). GLEIF publishes production witness, AID, and schema OOBIs at `https://gleif-it.github.io/.well-known/keri/oobi/`.

### Structure

The well-known directory follows a hierarchical structure:

```
/.well-known/
├── index.json                  # Main discovery index
├── schema.json                 # JSON Schema for validation
└── keri/oobi/
    ├── index.json              # OOBI catalog (AIDs, witnesses, schemas)
    ├── schema.json             # OOBI index schema
    └── {identifier}/           # Per-identifier directories
        └── index.json          # Resource content
```

### Discovery Protocol

**Step 1**: Discover available resources

```bash
curl https://gleif-it.github.io/.well-known/keri/oobi/index.json
```

Returns:

```json
{
  "$schema": "https://gleif-it.github.io/.well-known/keri/oobi/schema.json",
  "aids": {
    "GLEIF RoOT": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
    "GLEIF External": "EINmHd5g7iV-UldkkkKyBIH052bIyxZNBn9pq-zNrYoS"
  },
  "witnesses": {
    "BDkq35LUU63xnFmfhljYYRY0ymkCg7goyeCxN30tsvmS": "BDkq35LUU63xnFmfhljYYRY0ymkCg7goyeCxN30tsvmS"
  },
  "schemas": {
    "LegalEntityvLEICredential": "ENPXp1vQzRF6JwIuS-mp2U8Uf1MoADoP_GqQ62VsDZWY"
  }
}
```

**Step 2**: Resolve specific identifier

```bash
# Get AID witness URLs
curl https://gleif-it.github.io/.well-known/keri/oobi/EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2/index.json
```

Returns KERI `rpy` message with witness URLs:

```json
{
  "v": "KERI10JSON000282_",
  "t": "rpy",
  "r": "/oobi/witness",
  "a": {
    "urls": [
      "http://5.161.69.25:5623/oobi/EDP1vHcw_.../witness",
      "http://51.161.130.60:5623/oobi/EDP1vHcw_.../witness"
    ],
    "aid": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2"
  }
}
```

### Resource Types

Well-known endpoints serve three resource types:

| Type | Prefix | Content | Content-Type |
|------|--------|---------|--------------|
| **AID** | `E` | KERI rpy message with witness URLs | `application/cesr` |
| **Witness** | `B` | Witness KEL (icp + rpy messages) | `application/cesr` |
| **Schema** | `E` | ACDC JSON Schema | `application/cesr` |

### Configuration with wurls

Use well-known URIs in configuration files:

```json
{
  "dt": "2026-01-21T00:00:00.000000+00:00",
  "wurls": [
    "https://gleif-it.github.io/.well-known/keri/oobi/EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2"
  ]
}
```

The `wurls` field is processed by `Habery.reconfigure()` similar to `iurls` and `durls`.

### Implementation

The vLEI project includes a reference implementation in `src/vlei/app/well_known.py`:

```python
from vlei.app.well_known import loadWellKnownEnds
import falcon

app = falcon.App()
oobi_dir = "./samples/oobis/.well-known/keri/oobi"
loadWellKnownEnds(app, oobi_dir)
```

The handler:
1. Loads `index.json` to categorize identifiers
2. Routes requests based on identifier type (AID/witness/schema)
3. Returns appropriate content with correct content-type headers

Sample data from GLEIF's production well-known directory is available in `vLEI/samples/oobis/.well-known/` including 3 AIDs, 10 witnesses, and 8 vLEI credential schemas.

### Benefits

- **Standardized Discovery**: RFC 8615 compliant path structure
- **No Authentication**: Public discovery endpoints
- **Complete Resource Sets**: AIDs return all witness URLs in single response
- **Schema Resolution**: Direct access to ACDC schemas without OOBI resolution
- **Production Ready**: GLEIF operates this at scale for vLEI ecosystem

## Part 9: Configuration Best Practices

### For KERIpy Controllers

1. **Use configuration files for static topology**
   - Witnesses that rarely change
   - Well-known schema OOBIs
   - Persistent peer relationships

2. **Use OOBI resolution for dynamic discovery**
   - New witnesses discovered at runtime
   - Peer identifiers learned through introductions
   - Ephemeral connections

3. **Use endpoint role authorization for services**
   - Your own controller endpoints
   - Agent mailbox endpoints
   - Service availability changes

### For KERIA Agencies

1. **Set global `iurls` in agency config**
   - Standard witnesses all agents use
   - Common watcher pools
   - Shared schema registries

2. **Keep `curls` agent-specific**
   - Each agent may have unique controller endpoints
   - Avoid global `curls` unless all agents share endpoints

3. **Use configuration files for production**
   - Easier to manage than environment variables
   - Version controlled configuration
   - Clear separation of concerns

4. **Provide OOBI resolution API for runtime changes**
   - New witnesses can be added without restart
   - Dynamic peer discovery
   - Schema updates


## Conclusion

KERI configuration operates at multiple levels:

1. **Controller Configuration**: File-based `curls`, `iurls`, `durls` or OOBI resolution
2. **Witness Configuration**: Endpoint role authorization for witness services
3. **Mailbox Configuration**: Endpoint role authorization for agent/mailbox services
4. **Agency Configuration**: Global KERIA service settings and OOBIs
5. **Agent Configuration**: Per-controller settings derived from agency config
6. **Well-Known Discovery**: RFC 8615 compliant `/.well-known/keri/oobi/` endpoints for standardized resource discovery

Understanding these levels clarifies the apparent confusion about "no way to provide iurls except OOBIs." The truth is:

- **KERIpy**: Multiple methods exist (file, OOBI, endpoint messages, well-known URIs)
- **KERIA**: Agency provides global `iurls`, agents inherit them, runtime updates via OOBI resolution

Both approaches ultimately create the same database records: endpoint role authorizations (`db.ends`) and location schemes (`db.locs`). The choice between file-based configuration, OOBI resolution, and well-known discovery depends on your use case: static infrastructure, dynamic topology, or public discovery.

For production deployments, use:
- **Well-known URIs** for public discovery of organizational infrastructure
- **Agency config file** for stable, shared witnesses and watchers
- **OOBI resolution API** for dynamic peer discovery and new witnesses
- **Endpoint role messages** for your own service availability

This provides the flexibility to handle static infrastructure, dynamic peer-to-peer discovery, and standardized public resource publication.

## Further Reading

- [KERI Configuration Files in keripy](https://github.com/WebOfTrust/keripy/blob/main/src/keri/app/configing.py)
- [KERIA Agency and Agent Management](https://github.com/WebOfTrust/keria/blob/main/src/keria/app/agenting.py)
- [KERI Endpoint Role Authorization Specification](https://github.com/WebOfTrust/keripy/blob/main/src/keri/core/coring.py)
- [OOBI Specification](https://github.com/WebOfTrust/keripy/blob/main/src/keri/app/oobiing.py)
- [GLEIF Well-Known URI Structure Specification](https://github.com/GLEIF-IT/GLEIF-IT.github.io/blob/main/.well-known/STRUCTURE.md)
- [GLEIF Well-Known Schema Specification](https://github.com/GLEIF-IT/GLEIF-IT.github.io/blob/main/.well-known/SCHEMA.md)
- [GLEIF Production Well-Known Directory](https://gleif-it.github.io/.well-known/)
- [RFC 8615: Well-Known URIs](https://www.rfc-editor.org/rfc/rfc8615.html)
