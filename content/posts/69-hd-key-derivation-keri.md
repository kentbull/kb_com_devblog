+++
title = "Hierarchical Deterministic Key Derivation in KERI: A Deep Dive into SaltyCreator"
slug = "keri-hd-key-derivation-deep-dive"
date = "2026-01-22"

[taxonomies]
tags = ["keri", "keripy", "keria", "signify", "signify-ts", "cryptography", "key-management", "key derivation"]

[extra]
comment = true
+++

Understanding how cryptographic keys are deterministically derived in KERI is essential for anyone building or integrating with KERI-based systems. This post provides a comprehensive breakdown of the hierarchical deterministic (HD) key derivation algorithm implemented in KERIpy and SignifyTS, explaining how a simple passcode (the "bran") is the mechanism controlling reproducible generation, storage, encryption, and decryption of an infinite sequence of cryptographic keys. Along with this passcode a cryptographic salt and deterministic key paths are the heart of the KERI HD key generation scheme.

## Overview: The Key Derivation Pipeline

KERI uses two related but distinct key derivation processes, one for the encryption and decryption key and the other for the path-based derivation of private and public key pairs. Understanding this separation is critical:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    KEYSTORE INITIALIZATION (kli init)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│   ENCRYPTION/DECRYPTION KEY         KEYS USED FOR IDENTIFIERS               │
│   User Passcode (21+ chars)              Keystore Salt                      │
│         │                                (--salt or auto-generated)         │
│         ▼                                      │                            │
│   ┌─────────────┐               first 21 chars │                            │
│   │    Bran     │  "0A" + "A" + passcode[0:21] │                            │
│   └──────┬──────┘        thisismysecretkeyseed │                            │
│          │                                     │                            │
│          ▼                                     │                            │
│   ┌─────────────┐                              │                            │
│   │   Salter    │  + empty path ("")           │                            │
│   │             │  + tier                      │                            │
│   └──────┬──────┘                              │                            │
│          │                                     │                            │
│          ▼                                     │                            │
│   ┌─────────────┐                              ▼                            │
│   │    AEID     │  Non-transferable     ┌─────────────┐                     │
│   │   Signer    │  identifier for   ───►│  Encrypted  │                     │
│   │             │  encrypt/decrypt      │   Storage   │                     │
│   └─────────────┘                       │ (ks/gbls/   │                     │
│                                         │    salt)    │                     │
│                                         └─────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                    AID KEY DERIVATION (kli incept/rotate)                   │
├─────────────────────────────────────────────────────────────────────────────┤
│   KEYS USED FOR IDENTIFIERS                                                 │
│   Keystore Salt (decrypted using AEID from bran)                            │
│         │                                                                   │
│         ▼                                                                   │
│   ┌─────────────┐                                                           │
│   │   Salter    │  Base64 decode → 16 raw bytes                             │
│   │ (128-bit)   │  Used as Argon2id salt parameter (crypto_pwhash function) │
│   └──────┬──────┘                                                           │
│          │                                                                  │
│          │  + Path (stem + ridx + kidx)                                     │
│          │  + Security Tier (low/med/high)                                  │
│          ▼                                                                  │
│   ┌─────────────┐                                                           │
│   │  Argon2id   │  Stretches path + salt → 32-byte seed                     │
│   │  Stretch    │  Time/memory cost set by tier                             │
│   └──────┬──────┘                                                           │
│          │                                                                  │
│          ▼                                                                  │
│   ┌─────────────┐                                                           │
│   │   Signer    │  Ed25519 keypair from 32-byte seed                        │
│   │  (Keypair)  │  Contains: private key (seed) + public key (verfer)       │
│   └─────────────┘                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Key insight:** The user's passcode (bran) is **never** used directly for AID key derivation. It only creates the AEID used to encrypt/decrypt the keystore salt. The actual AID keys come from the keystore salt combined with derivation paths.

## The Bran: Master Secret and encryption key

The "bran" is a subset (up to the first 21 chars) of the user-provided "passcode" string along with the "0AA" prefix explained below. The user-provided portion of the bran is often labeled as "passcode" in the KERI ecosystem. While the bran is your master secret, it does **not** directly derive AID keys. Instead, it creates the AEID (Authentication and Encryption ID) used to protect the keystore salt, which in turn derives the actual AID keys.

A bran is composed of, in order, a type code on the front (`"0A"`), a filler "zero" character (`"A"`), and the first twenty one characters of the user-provided passcode string. Concatenating all these parts together forms a string like `"0AAthisismysecretkeyseed"` and constitutes a valid [CESR][CESR] cryptographic key seed. The construction process shown below is used to create a CESR data structure, specifically a cryptographic seed.

### Bran Construction

From KERIpy:
```python
bran = coring.MtrDex.Salt_128 + 'A' + bran[:21]  # qb64 salt for seed
# Results in: "0A" + "A" + "thisismysecretkeyseed"
#    = "0AAthisismysecretkeyseed"
```

And from SignifyTS:

```typescript
// SignifyTS - manager.ts
const bran = MtrDex.Salt_128 + 'A' + passcode.substring(0, 21);
// Results in: "0A" + "A" + "thisismysecretkeyseed"
//         = "0AAthisismysecretkeyseed"
```

The bran is constructed as a valid qb64-encoded CESR primitive:
- `0A` - The CESR code for a 128-bit salt
- `A` - Padding character for proper Base64 alignment
- First 21 characters of the passcode - The actual entropy

**Important**: The passcode must be at least 21 characters long to provide sufficient entropy (approximately 126 bits if using alphanumeric characters).

The reason the type code "0A" is prefixed on the front of the cryptographic seed CESR primitive is because CESR is a [type-length-value][TLV] data encoding format where the type code comes first in the order of bytes.

## The Salter: Managing Your Seed and Salt

The `Salter` class wraps the seed generated from the bran as a CESR encoded value and provides the Argon2id key stretching functionality. Within the KERIpy and SignifyTS codebases this encoding is called "qualified Base64," or "qb64" for short. The `Salter` class is used in two contexts, as the container for the cryptographic seed used as the encryption key and as the container for the base seed used for AID key derivation.

1. **AEID creation:** Wraps the bran and stretches it with an empty path to create the AEID
2. **AID key derivation:** Wraps the keystore salt and stretches it with derivation paths to create AID keys

```python
# KERIpy - signing.py
class Salter(Matter):
    """
    Salter is Matter subclass to maintain random salt for secrets (private keys)
    Its 
      .raw is the bytes of the salt, such as "0AAthisismysecretkeyseed"
      .code as cipher suite for salt (algorithm).

    The qb64 encoded seed/salt string is encoded as bytes to get 16 raw bytes for the raw argument.
    """
    def __init__(self, raw=None, code=MtrDex.Salt_128, tier=None, **kwa):
        super(Salter, self).__init__(raw=raw, code=code, **kwa)
        self.tier = tier if tier is not None else Tiers.low
```

The Salter receives the 16 raw bytes from the qb64-encoded seed/salt and stores the security tier for later stretching operations.

## The Path: Deterministic Key Indexing

The heart of the HD derivation is the **path** - a unique string that, when combined with the salt, produces a unique key. The path formula is:

```python
# KERIpy - keeping.py, SaltyCreator.create()
stem = self.stem if self.stem else "{:x}".format(pidx)  # hex pidx if no stem
path = "{}{:x}{:x}".format(stem, ridx, kidx + i)
```

### Path Components

The starting rotation index (ridx) and key index (kidx) are derivable from the current state of the key event log (KEL).

| Component | Description                                 | Format                | Example                         |
|-----------|---------------------------------------------|-----------------------|---------------------------------|
| `stem`    | Prefix identifier (or pidx in hex)          | string or hex         | `"signify:controller"` or `"0"` |
| `ridx`    | Rotation index (establishment event number) | hex (no padding)      | `0`, `1`, `a`, `10`             |
| `kidx`    | Key index (cumulative key count)            | hex (no padding)      | `0`, `1`, `2`, `a`              |
| `i`       | Index within current key set                | integer added to kidx | `0`, `1`, `2`                   |

#### Stem

`stem` is defaulted in KERIpy as follows from the name and namespace, if any:
```python
stem = self.name if self.ns is None else f"{self.ns}{self.name}"
```

The stem is a prefix to a path that may be used to namespace a given set of keys. For example, KERIpy uses the default stem of the string `"0"` for keys generated for keys stored in a KERIpy keystore. KERIpy also uses the SignifyTS uses the default stem of `"signify:controller"` for the Signify Controller and `"signify:aid"` for generated keys used for AIDs stored in KERIA.

### Path Examples

For a single-sig identifier with `pidx=0` (so stem becomes `"0"`):

| Event             | ridx | kidx | i | Path      | Description                      | Decimal kidx |
|-------------------|------|------|---|-----------|----------------------------------|--------------|
| Inception signing | 0    | 0    | 0 | `"000"`   | stem="0" + ridx="0" + kidx="0"   |            0 |
| Inception next    | 1    | 1    | 0 | `"011"`   | stem="0" + ridx="1" + kidx="1"   |            1 |
| Rotation 1 next   | 2    | 2    | 0 | `"022"`   | stem="0" + ridx="2" + kidx="2"   |            2 |
| Rotation 2 next   | 3    | 3    | 0 | `"033"`   | stem="0" + ridx="3" + kidx="3"   |            3 |
| Rotation 9 next   | a    | a    | 0 | `"0aa"`   | stem="0" + ridx="a" + kidx="a"   |            9 |
| Rotation 15 next  | 10   | 10   | 0 | `"01010"` | stem="0" + ridx="10" + kidx="10" |           16 |

Note: At each rotation, the previous "next" key becomes the current signing key (it was pre-generated).

## Key Generation Across 30 Keys: Visual Reference

Let's visualize the first 30 keys generated for a single-sig identifier with `pidx=0` (stem becomes `"0"`):

**Important**: At each establishment event, we generate:
1. **Signing keys** at `(ridx, kidx)` - these are used immediately
2. **Next keys** at `(ridx+1, kidx+count)` - pre-committed for the *next* rotation

At rotation, the previous "next" keys become the new signing keys (no re-generation needed), and we only generate NEW next keys.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SINGLE-SIG KEY SEQUENCE (pidx=0)                        │
│                         stem = "0" (pidx in hex)                                │
│                         Both ridx and kidx shown in HEX (decimal in parens)     │
├─────────────────────────────────────────────────────────────────────────────────┤
│ PATH     ridx    kidx    WHEN GENERATED        WHEN USED AS SIGNING KEY         │
├─────────────────────────────────────────────────────────────────────────────────┤
│ "000"     0       0      Inception             Inception (current signing)      │
│ "011"     1       1      Inception             Rotation 1 (becomes current)     │
│ "022"     2       2      Rotation 1            Rotation 2 (becomes current)     │
│ "033"     3       3      Rotation 2            Rotation 3 (becomes current)     │
│ "044"     4       4      Rotation 3            Rotation 4 (becomes current)     │
│ "055"     5       5      Rotation 4            Rotation 5 (becomes current)     │
│ "066"     6       6      Rotation 5            Rotation 6 (becomes current)     │
│ "077"     7       7      Rotation 6            Rotation 7 (becomes current)     │
│ "088"     8       8      Rotation 7            Rotation 8 (becomes current)     │
│ "099"     9       9      Rotation 8            Rotation 9 (becomes current)     │
│ "0aa"     a (10)  a (10) Rotation 9            Rotation 10 (becomes current)    │
│ "0bb"     b (11)  b (11) Rotation 10           Rotation 11 (becomes current)    │
│ "0cc"     c (12)  c (12) Rotation 11           Rotation 12 (becomes current)    │
│ "0dd"     d (13)  d (13) Rotation 12           Rotation 13 (becomes current)    │
│ "0ee"     e (14)  e (14) Rotation 13           Rotation 14 (becomes current)    │
│ "0ff"     f (15)  f (15) Rotation 14           Rotation 15 (becomes current)    │
│ "01010"  10 (16) 10 (16) Rotation 15           Rotation 16 (becomes current)    │
│ "01111"  11 (17) 11 (17) Rotation 16           Rotation 17 (becomes current)    │
│ "01212"  12 (18) 12 (18) Rotation 17           Rotation 18 (becomes current)    │
│ "01313"  13 (19) 13 (19) Rotation 18           Rotation 19 (becomes current)    │
│ "01414"  14 (20) 14 (20) Rotation 19           Rotation 20 (becomes current)    │
│ "01515"  15 (21) 15 (21) Rotation 20           Rotation 21 (becomes current)    │
│ "01616"  16 (22) 16 (22) Rotation 21           Rotation 22 (becomes current)    │
│ "01717"  17 (23) 17 (23) Rotation 22           Rotation 23 (becomes current)    │
│ "01818"  18 (24) 18 (24) Rotation 23           Rotation 24 (becomes current)    │
│ "01919"  19 (25) 19 (25) Rotation 24           Rotation 25 (becomes current)    │
│ "01a1a"  1a (26) 1a (26) Rotation 25           Rotation 26 (becomes current)    │
│ "01b1b"  1b (27) 1b (27) Rotation 26           Rotation 27 (becomes current)    │
│ "01c1c"  1c (28) 1c (28) Rotation 27           Rotation 28 (becomes current)    │
│ "01d1d"  1d (29) 1d (29) Rotation 28           Rotation 29 (becomes current)    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Key observation for single key single-sig**: `ridx == kidx` always, because each rotation uses exactly 1 key.
The path formula `stem + ridx_hex + kidx_hex` produces symmetric patterns like `"0aa"` and `"01010"`.

### Path Structure Breakdown

Notice how the path evolves as indices grow:

#### Critical Insight: Paths Are Generated, Not Parsed

A key point from [GitHub Discussion #929][GHDiscuss929] is that the no-separator design guarantees **collision-free paths within a KEL**, but does NOT make paths parseable without context.

**The Problem**: Given just a path string like `"039"`, you cannot determine the components:
- Is it `stem="0"`, `ridx=3`, `kidx=9` (a 3-key multisig after 2 rotations)?
- Or `stem="0"`, `ridx=0`, `kidx=39` (a 57+ key multisig at inception)?
- Or `stem="03"`, `ridx=9`, `kidx=???` (different stem)?

**The Resolution**: You never need to PARSE a path backward. You always GENERATE paths forward from known context, specifically the KEL.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    PATH GENERATION vs PARSING                                │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✗ WRONG APPROACH: Try to parse "039" → (stem=?, ridx=?, kidx=?)             │
│                    This is AMBIGUOUS without context!                        │
│                                                                              │
│  ✓ RIGHT APPROACH: Generate path from known parameters (the KEL):            │
│                                                                              │
│    You have: stem="0" (from identifier params)                               │
│              ridx=3   (from KEL: this is rotation 2's next keys)             │
│              kidx=9   (from KEL: 3 keys × 3 events = 9)                      │
│              i=0      (first key in this set)                                │
│                                                                              │
│    Compute:  path = stem + hex(ridx) + hex(kidx + i)                         │
│                   = "0" + "3" + "9"                                          │
│                   = "039"                                                    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### What Context Is Required for Key Recovery?

To regenerate keys deterministically, you need:

| Context           | Source                                | Purpose                                      |
|-------------------|---------------------------------------|----------------------------------------------|
| `bran` (passcode) | User memory                           | Provides the salt for Argon2id               |
| `stem`            | Stored identifier parameters          | Path prefix (or derived from pidx)           |
| `tier`            | Stored identifier parameters          | Argon2id security level                      |
| `KEL`             | Retrieved from witnesses/KERI network | Provides ridx, kidx, and key count per event |

The generation algorithm walks the KEL sequentially:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    KEY GENERATION ALGORITHM (pseudocode)                     │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Given: bran, stem, tier, KEL                                                │
│                                                                              │
│  keys = [] # all keys in sequential order                                    │
│  kidx = 0                                                                    │
│  for each establishment_event in KEL:                                        │
│      ridx = event.sequence_number  (0 for inception, 1 for rot1, etc.)       │
│      key_count = len(event.keys)                                             │
│                                                                              │
│      for i in 0..key_count-1:                                                │
│          path = stem + hex(ridx) + hex(kidx + i)                             │
│          seed = argon2id(path, salt, tier)                                   │
│          keys[kidx] = ed25519_from_seed(seed)                                │
│                                                                              │
│      kidx += key_count    # Accumulate for next event                        │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Key Takeaway**: The no-separator path design works because you always have the KEL context when recovering keys. The mathematical proof in [Discussion #929][GHDiscuss929] ensures that within any valid KEL, no two establishment events can ever produce the same path - this prevents key collisions, not parsing ambiguity.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         PATH ANATOMY                                         │
│                   (all indices shown in HEX as used in path)                 │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Path "000"   → stem="0" + ridx="0"  + kidx="0"    (1+1+1 = 3 chars)        │
│   Path "0ff"   → stem="0" + ridx="f"  + kidx="f"    (1+1+1 = 3 chars)        │
│   Path "01010" → stem="0" + ridx="10" + kidx="10"   (1+2+2 = 5 chars)        │
│   Path "01f1f" → stem="0" + ridx="1f" + kidx="1f"   (1+2+2 = 5 chars)        │
│                                                                              │
│   ┌────────────┬────────────┬───────────┬────────────────────────────────┐   │
│   │ ridx (hex) │ kidx (hex) │   Path    │   Path Breakdown               │   │
│   ├────────────┼────────────┼───────────┼────────────────────────────────┤   │
│   │     0      │     0      │  "000"    │   "0" + "0" + "0"              │   │
│   │     9      │     9      │  "099"    │   "0" + "9" + "9"              │   │
│   │     a      │     a      │  "0aa"    │   "0" + "a" + "a"    (dec: 10) │   │
│   │     f      │     f      │  "0ff"    │   "0" + "f" + "f"    (dec: 15) │   │
│   │    10      │    10      │  "01010"  │   "0" + "10" + "10"  (dec: 16) │   │
│   │    ff      │    ff      │  "0ffff"  │   "0" + "ff" + "ff"  (dec: 255)│   │
│   │   100      │   100      │  "0100100"│   "0" + "100" + "100"(dec: 256)│   │
│   └────────────┴────────────┴───────────┴────────────────────────────────┘   │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Multi-Sig Key Generation

For a 2-of-3 multi-sig identifier, each establishment event generates 3 signing keys and 3 next keys. The `kidx` accumulates: at inception it's 0, after inception it becomes 3 (0 + 3 keys), after rotation 1 it becomes 6 (3 + 3 keys), etc.

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                      MULTI-SIG (2-of-3) KEY SEQUENCE                          │
│                      pidx=0, stem="0"                                         │
│                      ridx and kidx shown in HEX (decimal in parens)           │
├───────────────────────────────────────────────────────────────────────────────┤
│ PATH    ridx   kidx      i   WHEN GENERATED   ROLE                            │
├───────────────────────────────────────────────────────────────────────────────┤
│ "000"    0      0        0   Inception        signing key 1 (current)         │
│ "001"    0      0        1   Inception        signing key 2 (current)         │
│ "002"    0      0        2   Inception        signing key 3 (current)         │
│ "013"    1      3        0   Inception        next key 1 → Rot1 signing       │
│ "014"    1      3        1   Inception        next key 2 → Rot1 signing       │
│ "015"    1      3        2   Inception        next key 3 → Rot1 signing       │
│ ─────────────────────────── ROTATION 1 ───────────────────────────────────────│
│ "026"    2      6        0   Rotation 1       next key 1 → Rot2 signing       │
│ "027"    2      6        1   Rotation 1       next key 2 → Rot2 signing       │
│ "028"    2      6        2   Rotation 1       next key 3 → Rot2 signing       │
│ ─────────────────────────── ROTATION 2 ───────────────────────────────────────│
│ "039"    3      9        0   Rotation 2       next key 1 → Rot3 signing       │
│ "03a"    3      9        1   Rotation 2       next key 2 → Rot3 signing       │
│ "03b"    3      9        2   Rotation 2       next key 3 → Rot3 signing       │
│ ─────────────────────────── ROTATION 3 ───────────────────────────────────────│
│ "04c"    4      c (12)   0   Rotation 3       next key 1 → Rot4 signing       │
│ "04d"    4      c (12)   1   Rotation 3       next key 2 → Rot4 signing       │
│ "04e"    4      c (12)   2   Rotation 3       next key 3 → Rot4 signing       │
└───────────────────────────────────────────────────────────────────────────────┘
```

**Path formula reminder**: `path = stem + ridx_hex + (kidx + i)_hex`

For "04c": stem="0" + ridx="4" + (kidx=c + i=0) = "0" + "4" + "c" = "04c"

### Understanding kidx Accumulation

The key index (`kidx`) grows as keys are generated:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      KIDX ACCUMULATION (3-key multi-sig)                    │
│                      (showing decimal values, hex in parens where different)│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Inception:                                                                 │
│    Signing keys: ridx=0, kidx=0  (generates keys at paths 000, 001, 002)    │
│    Next keys:    ridx=1, kidx=3  (generates keys at paths 013, 014, 015)    │
│                  └─────── kidx = 0 + 3 = 3                                  │
│                                                                             │
│  Rotation 1:                                                                │
│    Signing keys: ridx=1, kidx=3  (already generated at inception)           │
│    Next keys:    ridx=2, kidx=6  (generates keys at paths 026, 027, 028)    │
│                  └─────── kidx = 3 + 3 = 6                                  │
│                                                                             │
│  Rotation 2:                                                                │
│    Signing keys: ridx=2, kidx=6  (already generated at rotation 1)          │
│    Next keys:    ridx=3, kidx=9  (generates keys at paths 039, 03a, 03b)    │
│                  └─────── kidx = 6 + 3 = 9                                  │
│                                                                             │
│  Rotation 3:                                                                │
│    Signing keys: ridx=3, kidx=9  (already generated at rotation 2)          │
│    Next keys:    ridx=4, kidx=12 (hex: c)  (paths 04c, 04d, 04e)            │
│                  └─────── kidx = 9 + 3 = 12 (hex: c)                        │
│                                                                             │
│  Formula: next_kidx = current_kidx + number_of_keys_per_event               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Why No Delimiter Between ridx and kidx?

A common question is: "Without a delimiter between ridx and kidx in the path, won't there be ambiguity?" For example, couldn't path `"111"` mean either `ridx=1, kidx=11` or `ridx=11, kidx=1`?

The answer is **no**, and here's why (as explained in [GitHub Discussion #929][GHDiscuss929]):

### Mathematical Proof of Unambiguity

Given any valid Key Event Log (KEL), the following properties always hold:

1. **kidx >= ridx**: The key index is always greater than or equal to the rotation index
2. **kidx increases monotonically**: Each establishment event uses at least one new key
3. **ridx increases by exactly 1**: Each rotation increments ridx by 1

This means for any two valid partitions of the same hex string:
- If partition A has a longer ridx than partition B
- Then partition A must have a smaller kidx than partition B
- But this violates property 1 (kidx >= ridx)

**Example**: Can `"1a"` represent both `ridx=1, kidx=a(10)` AND `ridx=1a(26), kidx=empty`?
- First partition: ridx=1, kidx=10 ✓ (kidx > ridx)
- Second partition: ridx=26, kidx=0 ✗ (kidx < ridx, impossible!)

This elegant mathematical property allows KERI to use the **minimally sufficient** path format without delimiters, following KERI's design principle of minimal sufficiency.

## The Stretching Function: Argon2id

The path and salt are combined using the Argon2id key derivation function in Salter.stretch() shown below.
The Salt used in Salter is randomly generated when a keystore is first created using `kli init` or a call to the `Habery` constructor in KERIpy.

```python
# KERIpy - signing.py
class Salter(Matter):
    ...
    def stretch(self, *, size=32, path="", tier=None, temp=False):
        """
        Returns raw binary seed derived from path and .raw (salt)
        stretched to size using argon2id stretching algorithm.
        """
        tier = tier if tier is not None else self.tier

        # Security parameters based on tier
        if temp:
            opslimit = 1  # pysodium.crypto_pwhash_OPSLIMIT_MIN
            memlimit = 8192  # pysodium.crypto_pwhash_MEMLIMIT_MIN
        else:
            if tier == Tiers.low:
                opslimit = 2  # pysodium.crypto_pwhash_OPSLIMIT_INTERACTIVE
                memlimit = 67108864  # pysodium.crypto_pwhash_MEMLIMIT_INTERACTIVE
            elif tier == Tiers.med:
                opslimit = 3  # pysodium.crypto_pwhash_OPSLIMIT_MODERATE
                memlimit = 268435456  # pysodium.crypto_pwhash_MEMLIMIT_MODERATE
            elif tier == Tiers.high:
                opslimit = 4  # pysodium.crypto_pwhash_OPSLIMIT_SENSITIVE
                memlimit = 1073741824  # pysodium.crypto_pwhash_MEMLIMIT_SENSITIVE
            else:
                raise ValueError("Unsupported security tier = {}.".format(tier))

        # stretch algorithm is argon2id - password hash algo used as a stretcher
        seed = pysodium.crypto_pwhash(
            outlen=size,       # 32 for Ed25516
            passwd=path,       # 000 for first key of single sig AID
            salt=self.raw,     # 0ADOuCna7ifKHklxC7cU0s2E - generated or passed in --salt arg
            opslimit=opslimit, # 3 for medium
            memlimit=memlimit, # 268435456 for medium
            alg=pysodium.crypto_pwhash_ALG_ARGON2ID13)
        return (seed)
```

### Security Tiers

| Tier   | opslimit | memlimit | Use Case                            |
|--------|----------|----------|-------------------------------------|
| `low`  | 2        | 64 MB    | Interactive applications            |
| `med`  | 3        | 256 MB   | Moderate security needs             |
| `high` | 4        | 1 GB     | High-security environments          |
| `temp` | 1        | 8 KB     | Testing only (NEVER in production!) |

## Code Walkthrough

Let's trace through the actual implementation for generating a key based on the deterministic (salty - using a salt) derivation strategy.

The general flow for the KLI in KERIpy is this:

### Keystore Initialization - setting the seed and salt
- A passcode and a salt may both be specified to `kli init`. Not specifying a `--salt` means one will be auto-generated.
- **Two distinct secrets are established:**
  1. **AEID (from bran):** The passcode is passed to the `bran` argument → `Habery.__init__` → `Habery.setup`. The bran is truncated to 21 chars and prefixed with `0AA` to form a valid CESR seed. This is stretched with an **empty path** (`""`) to create a `Signer` whose:
     - Private key becomes the `seed` for the keystore `Manager`
     - Public key becomes the `aeid` (Authentication and Encryption AID) used to encrypt/decrypt secrets stored in the keystore
  2. **Keystore Salt (from `--salt` or auto-generated):** A separate salt is either provided via `--salt` or randomly generated. This salt is stored (encrypted with the AEID) in the `ks/gbls/salt` LMDB key/value pair. **This is the salt used for actual AID key derivation.**

- **Important distinction:** The bran/passcode is **only** used to create the AEID for encryption. It is **not** used directly for AID key derivation. The stored keystore salt is what gets combined with derivation paths to generate AID keys.

### Key Generation for Inception or Rotation

- When `kli incept` (or `kli rotate`) is called, the bran is used to **unlock** the keystore, not to derive keys directly:
  1. The `bran` is passed to `existing.setupHby` → `Habery.__init__` → `Habery.setup`
  2. The bran is stretched (with empty path) to recreate the AEID
  3. The AEID decrypts the stored keystore salt from `ks/gbls/salt`
  4. The **decrypted keystore salt** (not the bran!) is passed to `SaltyCreator`
- Then during incept, `Habery.makeHab` calls `Hab.make` which adds a namespace/stem formatted like `<namespace><name>`, and calls `Manager.incept`
- `Manager.incept` calls `SaltyCreator.create` with the **keystore salt**, stem, and tier
- `SaltyCreator.create` generates the required number of keys using the derivation path:
  - Using `Salter.signer`, a path (`stem + ridx_hex + kidx_hex`) and tier are specified
  - The path is combined with the **keystore salt's raw bytes** and stretched via Argon2id
  - The output is the private key seed for each key
  - This is repeated for each key in the set (one per path)

### KERIpy Implementation

The code below shows the flow from the perspective of SaltyCreator having its `create` function invoked. It illustrates how the Salter is used to create a Signer by using the salt bytes, path, and tier with the Argon2id hash function to convert, or stretch, all those arguments into the set of bytes that is the private key, labeled here as `seed`.

```python
# keri/app/keeping.py
class SaltyCreator(Creator):
    def __init__(self, salt=None, stem=None, tier=None, **kwa):
        super(SaltyCreator, self).__init__(**kwa)
        self.salter = core.Salter(qb64=salt, tier=tier)
        self._stem = stem if stem is not None else ''

    def create(self, codes=None, count=1, code=MtrDex.Ed25519_Seed,
               pidx=0, ridx=0, kidx=0, transferable=True, temp=False, **kwa):
        signers = []
        if not codes:
            codes = [code for i in range(count)]

        # Determine stem: use provided stem or pidx as hex
        stem = self.stem if self.stem else "{:x}".format(pidx)
        
        for i, code in enumerate(codes):
            # Construct the path: stem + ridx_hex + kidx_hex
            path = "{}{:x}{:x}".format(stem, ridx, kidx + i)
            
            # Generate signer using Argon2id stretching
            signers.append(self.salter.signer(
                path=path,
                code=code,
                transferable=transferable,
                tier=self.tier,
                temp=temp
            ))
        return signers

# keri/core/signing.py
class Salter(Matter):
    ...
    def signer(self, *, code=MtrDex.Ed25519_Seed, transferable=True, path="",
               tier=None, temp=False):
        """
        Stretches the self.raw-salted derivation path into a proper length 
        key seed and returns a Signer instance that contains the keypair.
        """
        seed = self.stretch(
            size=Matter._rawSize(code), 
            path=path, 
            tier=tier, 
            temp=temp)
        return (Signer(raw=seed, code=code, transferable=transferable))

    def stretch(self, *, size=32, path="", tier=None, temp=False):
        """
        Returns raw binary seed derived from path and .raw (salt)
        stretched to size using argon2id stretching algorithm.
        """
        tier = tier if tier is not None else self.tier

        # Security parameters based on tier
        if temp:
            opslimit = 1  # pysodium.crypto_pwhash_OPSLIMIT_MIN
            memlimit = 8192  # pysodium.crypto_pwhash_MEMLIMIT_MIN
        else:
            if tier == Tiers.low:
                opslimit = 2  # pysodium.crypto_pwhash_OPSLIMIT_INTERACTIVE
                memlimit = 67108864  # pysodium.crypto_pwhash_MEMLIMIT_INTERACTIVE
            elif tier == Tiers.med:
                opslimit = 3  # pysodium.crypto_pwhash_OPSLIMIT_MODERATE
                memlimit = 268435456  # pysodium.crypto_pwhash_MEMLIMIT_MODERATE
            elif tier == Tiers.high:
                opslimit = 4  # pysodium.crypto_pwhash_OPSLIMIT_SENSITIVE
                memlimit = 1073741824  # pysodium.crypto_pwhash_MEMLIMIT_SENSITIVE
            else:
                raise ValueError("Unsupported security tier = {}.".format(tier))

        # stretch algorithm is argon2id - password hash algo used as a stretcher
        seed = pysodium.crypto_pwhash(
            outlen=size,       # 32 for Ed25516
            passwd=path,       # 000 for first key of single sig AID
            salt=self.raw,     # 0ADOuCna7ifKHklxC7cU0s2E
            opslimit=opslimit, # 3 for medium
            memlimit=memlimit, # 268435456 for medium
            alg=pysodium.crypto_pwhash_ALG_ARGON2ID13)
        return (seed)
```

### SignifyTS Implementation

The SignifyTS implementation is intentionally very similar to the Python implementation. There is one non-critical diference, noted below, that will likely be rectified in the near future.

```typescript
// keri/core/manager.ts
export class SaltyCreator implements Creator {
    public salter: Salter;
    private readonly _stem: string;
    
    constructor(salt?: string, tier?: Tier, stem?: string) {
        this.salter = new Salter({ qb64: salt, tier: tier });
        this._stem = stem == undefined ? '' : stem;
    }

    create(
        codes?: Array<string>,
        count: number = 1,
        code: string = MtrDex.Ed25519_Seed,
        transferable: boolean = true,
        pidx: number = 0,
        ridx: number = 0,
        kidx: number = 0,
        temp: boolean = false
    ): Keys {
        const signers = new Array<Signer>();
        const paths = new Array<string>();

        if (codes == undefined) {
            codes = new Array<string>(count).fill(code);
        }

        codes.forEach((code, idx) => {
            // Path construction differs based on stem presence
            const path = this.stem == ''
                ? pidx.toString(16)
                : this.stem + ridx.toString(16) + (kidx + idx).toString(16);

            signers.push(
                this.salter.signer(code, transferable, path, this.tier, temp)
            );
            paths.push(path);
        });

        return new Keys(signers, paths);
    }
}
```

### Implementation Difference: KERIpy vs SignifyTS

As of January 2026 there's an important implementation difference between the two codebases that will likely be aligned in the near future.

**KERIpy** (canonical):
```python
stem = self.stem if self.stem else "{:x}".format(pidx)
path = "{}{:x}{:x}".format(stem, ridx, kidx + i)
# Empty stem → path = pidx_hex + ridx_hex + kidx_hex
# With stem  → path = stem + ridx_hex + kidx_hex
```

**SignifyTS** (current):
```typescript
const path = this.stem == ''
    ? pidx.toString(16)  // ONLY pidx!
    : this.stem + ridx.toString(16) + (kidx + idx).toString(16);
// Empty stem → path = pidx_hex (missing ridx, kidx!)
// With stem  → path = stem + ridx_hex + kidx_hex
```

This means **SignifyTS with an empty stem will NOT generate the same keys as KERIpy** for the same parameters. In SignifyTS when stem is empty, all keys for the same `pidx` would have the same path and thus the same key - which is clearly incorrect for multi-key scenarios.

In practice, SignifyTS typically uses a non-empty stem (like `"signify:controller"`) which makes this difference moot, but it's important to be aware of when implementing cross-platform recovery or interoperability.

## Recovery: Regenerating Keys

One of the most powerful features of this HD derivation scheme is **recovery**. Given:
1. The original passcode (bran) - to decrypt the keystore
2. The keystore data (containing the encrypted salt) - or a backup of the salt
3. The Key Event Log (KEL) - to determine path parameters

You can regenerate all private keys by:
1. Using the bran to recreate the AEID and decrypt the keystore salt
2. Parsing the KEL to determine ridx and kidx values for each establishment event
3. Using the known stem (or pidx) from the identifier parameters
4. Reconstructing each path and re-stretching with the keystore salt to get the original keys

**Note:** If the keystore is lost and no salt backup exists, recovery is impossible even with the correct bran - because the salt used for key derivation is separate from the bran.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KEY RECOVERY PROCESS                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────┐     ┌───────────┐     ┌──────────┐                           │
│   │ Passcode │     │ Keystore  │     │   KEL    │                           │
│   │  (bran)  │     │  (salt)   │     │ (events) │                           │
│   └────┬─────┘     └─────┬─────┘     └────┬─────┘                           │
│        │                 │                │                                 │
│        ▼                 │                │                                 │
│   ┌─────────────┐        │                │                                 │
│   │ Recreate    │        │                │                                 │
│   │ AEID        │        │                │                                 │
│   └──────┬──────┘        │                │                                 │
│          │               │                │                                 │
│          ▼               │                ▼                                 │
│   ┌─────────────────┐    │         ┌──────────────┐                         │
│   │ Decrypt stored  │◄───┘         │ Parse Events │                         │
│   │ keystore salt   │              │ Extract:     │                         │
│   └────────┬────────┘              │ - ridx       │                         │
│            │                       │ - kidx       │                         │
│            │                       │ - stem/pidx  │                         │
│            │                       └──────┬───────┘                         │
│            │                              │                                 │
│            ▼                              ▼                                 │
│   ┌──────────────────────────────────────────┐                              │
│   │    Reconstruct Each Path                 │                              │
│   │    path = stem + ridx_hex + kidx_hex     │                              │
│   └────────────────────┬─────────────────────┘                              │
│                        │                                                    │
│                        ▼                                                    │
│   ┌──────────────────────────────────────────┐                              │
│   │   Argon2id Stretch Each                  │                              │
│   │   seed = stretch(path, keystore_salt)    │                              │
│   └────────────────────┬─────────────────────┘                              │
│                        │                                                    │
│                        ▼                                                    │
│   ┌──────────────────────────────────────────┐                              │
│   │   Recovered Private Keys                 │                              │
│   │   (identical to originals)               │                              │
│   └──────────────────────────────────────────┘                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Summary Table: Key Derivation Parameters

| Parameter | Source                              | Purpose                                       |
|-----------|-------------------------------------|-----------------------------------------------|
| `bran`    | User passcode (21+ chars)           | Creates AEID for keystore encryption          |
| `salt`    | `--salt` arg or auto-generated      | Argon2id salt for AID key derivation          |
| `aeid`    | Derived from bran (empty path)      | Encrypts/decrypts secrets in keystore         |
| `tier`    | Configuration                       | Controls Argon2id time/memory                 |
| `pidx`    | Manager state                       | Prefix index (identifier sequence)            |
| `stem`    | Per-identifier config               | Path prefix (or pidx if empty)                |
| `ridx`    | KEL state                           | Rotation index (establishment event count)    |
| `kidx`    | KEL state                           | Key index (cumulative key count)              |
| `i`       | Loop variable                       | Index within current key set                  |
| `path`    | Computed                            | `stem + ridx_hex + kidx_hex`                  |
| `seed`    | Argon2id output                     | 32-byte Ed25519 private key                   |

## Conclusion

The KERI HD key derivation algorithm elegantly combines:
- **User-friendly input**: A memorable passcode
- **Strong cryptography**: Argon2id key stretching
- **Deterministic paths**: Unambiguous ridx/kidx indexing
- **Full recoverability**: All keys reproducible from passcode + KEL

Understanding this algorithm is crucial for:
- Implementing KERI-compatible wallets
- Building recovery mechanisms
- Auditing key management security
- Debugging key generation issues

The mathematical elegance of the delimiter-free path format demonstrates KERI's commitment to minimal sufficiency while maintaining cryptographic rigor.

## References

- [GitHub Discussion #929: Why the path in Salty Creator does not need a separator](https://github.com/WebOfTrust/keripy/discussions/929)
- [GitHub Discussion #927: Signify stem index delimiter](https://github.com/WebOfTrust/keripy/discussions/927)
- [KERIpy Source: keri/app/keeping.py](https://github.com/WebOfTrust/keripy/blob/master/src/keri/app/keeping.py)
- [SignifyTS Source: keri/core/manager.ts](https://github.com/WebOfTrust/signify-ts/blob/main/src/keri/core/manager.ts)
- [Argon2 Specification](https://github.com/P-H-C/phc-winner-argon2)


## Appendix: The Internal Notary (Signator)

A fascinating real-world application of the HD key derivation scheme in KERI is the `Signator` class. When you initialize a KERI environment (a `Habery`), the system automatically creates an internal, hidden identifier to act as a local notary.

### What is the Signator?
The `Signator` is a non-transferable (basic) AID that KERI uses for its own internal security architecture. It demonstrates how the `stem` parameter is used to isolate key-spaces.

*   **Reserved Alias (Stem):** It always uses the fixed alias `__signatory__` as its stem.
   * `stem` is defaulted in KERIpy as follows from the name and namespace, if any:
     ```python
     stem = self.name if self.ns is None else f"{self.ns}{self.name}"
     ```
*   **Hidden Presence:** It is marked as `hidden=True` and `transferable=False`. It doesn't show up in your list of AIDs and its keys never rotate.
*   **Derived from `bran`:** Its private key is derived from the same `bran` (passcode) you provide during `kli init`.

### Purpose: Security at Rest
The `Signator` signs internal system data to ensure its integrity and authenticity while stored "at rest":

1.  **BADA Data:** It signs data following the "Best Available Data Acceptance" model, ensuring that local database records haven't been tampered with.
2.  **Internal Notifications:** Local system notifications for the user are signed by the `Signator`.
3.  **Connection Challenges:** During OOBI resolution and connection establishment, the `Signator` signs the nonces and challenges used to authenticate the peer.

### Why This Matters for HD Schemes
The `Signator` is a perfect example of HD derivation in practice:

1.  **Isolation via Stemming:** Even though your primary AID and the `Signator` share the same `bran`, they are cryptographically isolated because their paths start with different stems (e.g., `stem="0"` vs `stem="__signatory__"`).
2.  **Automatic Recovery:** Because the `Signator` is derived deterministically, if you recover your wallet from your passcode on a new machine, the internal `Signator` AID is also automatically recovered. All previously signed internal data remains verifiable without you ever having to manually back up a "system key."


[CESR]: https://trustoverip.github.io/kswg-cesr-specification/
[GHDiscuss929]: https://github.com/WebOfTrust/keripy/discussions/929
[TLV]: https://grokipedia.com/page/Type%E2%80%93length%E2%80%93value