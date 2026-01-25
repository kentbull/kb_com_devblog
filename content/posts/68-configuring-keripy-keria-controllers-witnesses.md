+++
draft = true
title = "Configuring KERIpy Controllers, Witnesses, and KERIA Agents"
slug = "configuring-keripy-keria-controllers-witnesses"
date = "2026-01-21"

[taxonomies]
tags=["keri", "keria", "keripy", "configuration", "witnesses", "controllers", "agents", "iurls", "curls", "durls", "oobi"]

[extra]
comment = true
+++

# Configuring KERIpy Controllers, Witnesses, and KERIA Agents

How do you configure witnesses, mailboxes, and service endpoints in KERIpy and KERIA? This article explains the configuration mechanisms from file-based configuration to endpoint role authorization, clarifying the distinction between controller, witness, mailbox, agency, and agent configurations.

**Disclaimer**: This article assumes intermediate familiarity with KERI concepts including identifiers (AIDs), witnesses, OOBIs, and the difference between KERIpy (the KERI implementation) and KERIA (the agent service).

## Introduction: The Configuration Challenge

Configuring KERIpy controllers and KERIA agents occurs at multiple levels:

1. **Controller Configuration** (KERIpy) - How your local identifier finds witnesses and peers, and which ports it listens on, if any.
2. **Witness Configuration** (KERIpy) - The endpoints witnesses listen on
3. **Mailbox Configuration** (KERIpy) - Where receipts and messages are delivered to
4. **Agency Configuration** (KERIA) - Global settings for the KERIA service
5. **Agent Configuration** (KERIA) - Per-agent settings within KERIA

Let's explore each, starting with file-based configuration and progressing to runtime endpoint role authorization.

## Part 1: KERIpy Controller Configuration

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

**Configuration URL Types:**

- **`curls`** (Controller URLs): URLs + ports on which to set up either a TCP or HTTP port listener - where this controller can be reached
- **`iurls`** (Introduction URLs): OOBI URLs for witnesses, watchers, mailboxes, and any KERI controller to resolve at startup
- **`durls`** (Data URLs): OOBI URLs for static data like ACDC schemas or credentials
- **`wurls`** (Well Known URLs): OOBI URLs specifically for well known components

### Habery Configuration and Initialization

The `Habery` class in `keri/app/habbing.py` manages the configuration lifecycle:

```python
class Habery:
    def __init__(self, name="test", base="", bran=None, 
                 ks=None, db=None, cf=None, temp=False, 
                 salt=None, **kwa):
        """
        Parameters:
            name: Database name differentiator
            base: Additional path segment for hierarchy
            bran: 21-character passcode for keystore encryption
            ks: Optional pre-created Keeper (keystore)
            db: Optional pre-created Baser (database)
            cf: Optional pre-created Configer (config file)
            temp: Use temporary directory (cleared on close)
            salt: QB64 salt for deterministic key generation
        """
```

Three initialization patterns exist:

TODO really? Double check this. Maybe best to remove or condense this section.

#### Pattern 1: Auto-initialization (Simplest)

```python
with habbing.openHby(name="test", temp=True) as hby:
    hab = hby.makeHab(name="alice")
    # Config file auto-created at ~/.keri/cf/test.json
```

#### Pattern 2: Pre-configured with File

```python
# Create config file first
cf = configing.Configer(name="myconfig", base="mybase", temp=False)
conf = {
    "dt": help.nowIso8601(),
    "iurls": ["http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha/controller"],
}
cf.put(conf)

# Pass to Habery
hby = habbing.Habery(name="test", base="mybase", cf=cf, temp=False)
```

#### Pattern 3: Dependency Injection with Doers

For production systems with managed lifecycles:

```python
ks = keeping.Keeper(name="test", temp=False)
db = basing.Baser(name="test", temp=False)
cf = configing.Configer(name="test", temp=False)

# Configure
conf = {
    "dt": help.nowIso8601(),
    "iurls": ["http://127.0.0.1:5642/oobi/BBilc4-L3tFUnfM_wJr4S4OJanAv_VmF_dJNN6vkf2Ha/controller"]
}
cf.put(conf)

# Create Habery with injected dependencies
hby = habbing.Habery(name="test", ks=ks, db=db, cf=cf, temp=False)

# Wrap in Doers for lifecycle management
doers = [
    keeping.KeeperDoer(keeper=ks),
    basing.BaserDoer(baser=db),
    configing.ConfigerDoer(configer=cf),
    habbing.HaberyDoer(habery=hby)
]

# Run with Doist
doist = doing.Doist(limit=1.0, tock=0.03125, real=True)
doist.do(doers=doers)
```

### The reconfigure Method

Controllers can be reconfigured at runtime using `Habery.reconfigure()`:

```python
def reconfigure(self, name, temp=False, clear=False, **kwa):
    """
    Reconfigure Habery from its config file
    
    Reads configuration and processes:
    - curls: Controller endpoint OOBIs
    - iurls: Introduction OOBIs for witnesses/peers
    - durls: Data OOBIs for schemas/credentials
    
    Creates endpoint role authorizations and location schemes
    """
```

This method:

1. Reads the config file via `self.cf.get()`
2. Extracts any subsection matching the hab's name
3. Processes `curls` to create controller endpoint role authorizations
4. Processes `iurls` to resolve witness/peer OOBIs
5. Processes `durls` to resolve data OOBIs
6. Updates the database with endpoint and location records

Example from `test_habbing.py`:

```python
# Setup Tam's config
curls = ["tcp://localhost:5620/"]
iurls = ["tcp://localhost:5621/?role=peer&name=nel"]
conf = dict(
    dt=help.nowIso8601(), 
    tam=dict(dt=help.nowIso8601(), curls=curls), 
    iurls=iurls
)
tamHby.cf.put(conf)

# Create hab - automatically calls reconfigure
tamHab = tamHby.makeHab(name="tam", isith='1', icount=3, 
                        toad=2, wits=[wesHab.pre, wokHab.pre])

# Verify configuration was applied
ender = tamHab.db.ends.get(keys=(tamHab.pre, "controller", tamHab.pre))
assert ender.allowed
locer = tamHab.db.locs.get(keys=(tamHab.pre, kering.Schemes.tcp))
assert locer.url == 'tcp://localhost:5620/'
```

### Namespaced Habs

TODO Verify this. Really namespaced config? How so? Remove if not true.

KERIpy supports namespace isolation for multiple identifiers:

```python
# Default namespace
hab1 = hby.makeHab(name="alice")

# Agent namespace
hab2 = hby.makeHab(name="alice", ns="agent")

# Controller namespace  
hab3 = hby.makeHab(name="alice", ns="controller")

# Each has separate config subsections
conf = {
    "dt": "...",
    "alice": {"curls": [...]},  # Default namespace
    "agent.alice": {"curls": [...]},  # Agent namespace
    "controller.alice": {"curls": [...]}  # Controller namespace
}
```

Configuration subsections use the format `{namespace}.{name}` or just `{name}` for the default namespace.

## Part 2: Witness Configuration

Witnesses are non-transferable identifiers that provide receipt services. Their configuration differs from controllers:

### Witness Endpoint Configuration

Witnesses need two types of configuration:

1. **Controller endpoints** - Where the witness accepts events
2. **Location schemes** - The URLs where the witness is reachable

```python
# Create witness hab (non-transferable)
wesHab = wesHby.makeHab(name='wes', isith="1", icount=1, 
                        transferable=False)

# Configure witness endpoints
msgs = bytearray()
msgs.extend(wesHab.makeEndRole(
    eid=wesHab.pre,
    role=kering.Roles.controller,
    stamp=helping.nowIso8601()
))

msgs.extend(wesHab.makeLocScheme(
    url='http://127.0.0.1:8888',
    scheme=kering.Schemes.http,
    stamp=helping.nowIso8601()
))

wesHab.psr.parse(ims=bytearray(msgs))
```

### Witness Discovery

Controllers discover witnesses through:

1. **Configuration `iurls`**: Pre-configured witness OOBIs
2. **OOBI resolution**: Resolving witness OOBIs at runtime
3. **Well-known endpoints**: Using `.well-known/keri/oobi/` discovery

When you specify witnesses during inception:

```python
hab = hby.makeHab(name='alice', wits=[wesHab.pre, wokHab.pre], toad=2)
```

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

The config file follows the same structure as KERIpy controller configs:

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

### Agent Creation Flow

```python
def create(self, caid, salt=None):
    """Create new agent with configuration"""
    habName = f"agent-{caid}"
    ks = keeping.Keeper(name=caid, base=self.base, temp=self.temp, reopen=True)
    agent_cf = self._writeAgentConfig(caid)  # Write agent config
    
    # Create Habery with agent config
    agentHby = habbing.Habery(
        name=caid,
        base=self.base,
        bran=self.bran,
        ks=ks,
        cf=agent_cf,  # Agent-specific config
        temp=self.temp,
        salt=salt
    )
    
    # Create agent hab
    agentHab = agentHby.makeHab(habName, ns="agent", 
                                transferable=True, delpre=caid)
    
    agentRgy = Regery(hby=agentHby, name=agentHab.name, 
                     base=self.base, temp=self.temp)
    
    agent = Agent(hby=agentHby, rgy=agentRgy, agentHab=agentHab, 
                  caid=caid, agency=self)
    
    # Cache agent
    self.agents[caid] = agent
    self.extend([agent])
    
    return agent
```

### Agent Retrieval Flow

When retrieving an existing agent:

```python
def get(self, caid):
    """Retrieve agent (from cache or database)"""
    # Check cache
    if caid in self.agents:
        agent = self.agents[caid]
        agent.last = helping.nowUTC()
        return agent
    
    # Load from database
    aaid = self.adb.agnt.get(keys=(caid,))
    if aaid is None:
        return None
    
    ks = keeping.Keeper(name=caid, base=self.base, temp=self.temp, reopen=True)
    
    # Create Habery WITHOUT cf parameter
    # Config is loaded from existing file
    agentHby = habbing.Habery(
        name=caid, 
        base=self.base, 
        bran=self.bran, 
        ks=ks, 
        temp=self.temp
    )
    
    agentHab = agentHby.habByName(f"agent-{caid}", ns="agent")
    
    agentRgy = Regery(hby=agentHby, name=agentHab.name, 
                     base=self.base, temp=self.temp)
    
    agent = Agent(hby=agentHby, rgy=agentRgy, agentHab=agentHab, 
                  agency=self, caid=caid)
    
    self.agents[caid] = agent
    self.extend([agent])
    
    return agent
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
class Roles:
    witness = "witness"      # Witness service
    controller = "controller"  # Controller service
    agent = "agent"          # Agent service
    mailbox = "mailbox"      # Mailbox service
    peer = "peer"            # Peer service
    watcher = "watcher"      # Watcher service
```

**Available Schemes:**

```python
class Schemes:
    http = "http"
    https = "https"
    tcp = "tcp"
    udp = "udp"
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

## Part 8: Configuration Best Practices

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

### Configuration File Organization

Recommended structure for multi-agent systems:

```json
{
  "dt": "2026-01-21T00:00:00.000000+00:00",
  
  "keria": {
    "dt": "2026-01-21T00:00:00.000000+00:00",
    "curls": []
  },
  
  "iurls": [
    "http://witness1.example.com:5644/oobi/BBWmLeV...",
    "http://witness2.example.com:5644/oobi/BN8t3n1...",
    "http://watcher1.example.com:5645/oobi/BLFqR4t..."
  ],
  
  "durls": [
    "http://schemas.gleif.org/oobi/ENxQ5cL...",
    "http://schemas.example.com/oobi/EKzW9jP..."
  ]
}
```

## Conclusion

KERI configuration operates at multiple levels:

1. **Controller Configuration**: File-based `curls`, `iurls`, `durls` or OOBI resolution
2. **Witness Configuration**: Endpoint role authorization for witness services
3. **Mailbox Configuration**: Endpoint role authorization for agent/mailbox services
4. **Agency Configuration**: Global KERIA service settings and OOBIs
5. **Agent Configuration**: Per-controller settings derived from agency config

Understanding these levels clarifies the apparent confusion about "no way to provide iurls except OOBIs." The truth is:

- **KERIpy**: Multiple methods exist (file, OOBI, endpoint messages)
- **KERIA**: Agency provides global `iurls`, agents inherit them, runtime updates via OOBI resolution

Both approaches ultimately create the same database records: endpoint role authorizations (`db.ends`) and location schemes (`db.locs`). The choice between file-based configuration and OOBI resolution depends on your use case: static vs. dynamic topology.

For production KERIA deployments, use:
- **Agency config file** for stable, shared witnesses and watchers
- **OOBI resolution API** for dynamic peer discovery and new witnesses
- **Endpoint role messages** for your own service availability

This provides the flexibility to handle both static infrastructure and dynamic peer-to-peer discovery.

## Further Reading

- [KERI Configuration Files in keripy](https://github.com/WebOfTrust/keripy/blob/main/src/keri/app/configing.py)
- [KERIA Agency and Agent Management](https://github.com/WebOfTrust/keria/blob/main/src/keria/app/agenting.py)
- [KERI Endpoint Role Authorization Specification](https://github.com/WebOfTrust/keripy/blob/main/src/keri/core/coring.py)
- [OOBI Specification](https://github.com/WebOfTrust/keripy/blob/main/src/keri/app/oobiing.py)
