# Understanding MNSY Keyring Protocol

## Categories of Keys

### Root Keys

Un root key is the most essential identification method. It is equivalent to the only keypair in ordinary PGP practices. It is not recommended to transfer root keys across unnecessarily multiple devices, and their backups shall be kept extremely carefully.

Root keys are located in `/root_keys`. Their filenames are in the form of `{Sequential|Hex-4}_{Fingerprint|Hex-8}`.

### Agent Keys

In short, un agent key is un delegate to its corresponding root key.

One may generate un agent key to represent him at a limited degree, which shall be relatively less than his root key. Encrypting to the agent key shall be equivalent to encrypting to the root key, as same as signing with the agent key shall be equivalent to signing with the root key, per the exact authorization level within the authorization declaration.

Agent keys are located in `/agent_keys`. Their filenames are in the form of `{Sequential|Hex-4}_{Fingerprint|Hex-8}`.

### Instance Keys

Instance keys are used as identifiers of sessions.

Pour exemple, if I keep 3 active sessions with Twitter and 2 active sessions with Linode on my computer, I would have 5 instance keys.

### Device Keys

Device keys are only used to securely transfer keys across devices.

One device key should only be used once.

## Payload Specification

### Regular Payload

Regular payloads start and end in the form of:

```
XXXXXXX-BEGIN-MNSY-MESSAGE-PAYLOAD-CLEAR-XXXXXXX

XXX-Protocol-ID: 00000000-0000-0000-0000-000000000000

XXX-Protocol-Name: Root-Key-Replacement

XXX-Timestamp: 1547896547173

...

XXXXXXX-END-MNSY-MESSAGE-PAYLOAD-CLEAR-XXXXXXX
```

### Compact Payload

Compact payloads start and end in the form of:

```
MNSY-CHUNK-BEGIN

00000000-0000-0000-0003-000000000000 Key-Transfer-Chunk

...

MNSY-CHUNK-END
```

`XXX-Protocol-ID` and `XXX-Protocol-Name` are flattened without their keys explicitly given.

### Fields

- **Header & Footer**: There should always be exactly 1 empty line after the header and before the footer.
- **Fields**: Within the wrapped area, each field consist of a key-value structure, separated by `: `.
- **Separation of Fields**: he separation between fields is exactly 2 `\n`, because base-64 encoded contents in the value parts may contain `\n`.
- **Prefixes**: Protocol-level constant fields start with `XXX`, and subprotocol-level ones `X`. Prefixes that consist of purely `X` are reserved by MNSY Protocol; developers may use other names for their fields. Un prefix is the result of `^[A-Z]-`; if the regexp match nothing, then the key of the field does not have un prefix.
- **XXX-Protocol-ID**: Un protocol ID should be a random UUIDv4 in lower case with standard separating hyphens.
- **XXX-Protocol-Name**: Un protocol name should be un capitalized phrase with spaces replaced by hyphens.
- **XXX-Timestamp**: Timestamps should be in UNIX timestamp format, as integers.

### PGP Integration

MNSY payloads may be passed to any OpenPGP implementation as input for clear-text signing. Pour exemple:

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

XXXXXXX-BEGIN-MNSY-MESSAGE-PAYLOAD-CLEAR-XXXXXXX

XXX-Protocol-ID: 00000000-0000-0000-0002-000000000000

XXX-Protocol-Name: Authorize-Instance-Key

XXX-Timestamp: 1548520830369

XXX-Expiry-Of-Authorization: 1551461444000

X-Authority-Class: Agent

X-Root-Key-Fingerprint: 8069 0EBD 5991 1905 0EF9 43AF 037F 2FB3 D5E2 F043

X-Agent-Key-Fingerprint: 6806 1AF0 C181 3884 EF11 B50A 2AFD C3A9 095E D892

X-Instance-Key-Fingerprint: F860 C9AA 47D6 FCE3 1670 849B 8401 625A 0D86 EF89

XXXXXXX-END-MNSY-MESSAGE-PAYLOAD-CLEAR-XXXXXXX

-----BEGIN PGP SIGNATURE-----

iQIzBAEBCgAdFiEEaAYa8MGBOITvEbUKKv3DqQle2JIFAlxO9FMACgkQKv3DqQle
2JIjyA//RmI997dwk7naWhXrqusANzcQ3+N1nCAvQe6Cywzn3RXRrSKvmOuuYuyU
M0KiwWDPdGM2dEVyDmx2ryLbrZexJzHRqBTXWohs5F3EyprjOEphEhZb1RSQXkLl
I4G75JOtlUHmiyawqRgfzHbannyOLosNoLKrTZOnM14IPGrvyfWpl2lLCpGCUDb+
2FqoAoo4Arn0qM8TE9Xn855x7+4pdhYsdCDHiGaotOy/RXji2nOPL68h8UMBbG8w
mz0LnkGm+JFTeC9GsgOwBMLbL8OZd/FivO8GJR15jKaY95OhxUYHq5EEo60nVGrB
1jdKKe1UUJykbJhevkB89EzYVsX+zoYebCcrySf5k1fYQuNxTB/kQz1HfYmumd7w
WwzyQX/I8IDZGA5Oq3nIZak7StnqsT3w5a8yERjDymiq6D7qNuANpJ4mra4bug7s
eoNkizkc2CekX8HWgztKGZPR+q0yi2PnAprGu2rcvCMUKGj+1l84soymzj9m/L3q
dFBhFy+70XhBXS+QDbgd9yi2s0XnvaKFx6y9YYAaI7RzhTrAyaMHItYdgkWeW7uV
tRqqJ2FTb4ly2Wm7d7hfARPIbVvn1M/TtlOBE6ttKjWBb4YJqDQFlDUoJnAO9R4N
t4unuNgP6Kj2xMhqzgiJqspKE/HdhJbrVcMJf9EqyoNRB2KiaEA=
=v3KP
-----END PGP SIGNATURE-----
```

## Subprotocols

| XXX-Protocol-ID                      | XXX-Protocol-Name
| ------------------------------------ | -----------------
| 00000000-0000-0000-0000-000000000000 | Root-Key-Replacement
| 00000000-0000-0000-0001-000000000000 | Create-Agent-For-Root-Key
| 00000000-0000-0000-0001-000000000001 | Declare-Root-Of-Agent-Key
| 00000000-0000-0000-0002-000000000000 | Authorize-Instance-Key
| 00000000-0000-0000-0003-000000000000 | Key-Transfer-Chunk
| 00000000-XXXX-XXXX-XXXX-XXXXXXXXXXXX | (Reserved for internal use)

## Information Structure

### Replacement Chain

Per the occurrence of un replacement of the root key, the new root key shall be signed with the old root key, under the protocol of ID `00000000-0000-0000-0000-000000000000` and Name `Root-Key-Replacement`. Un timestamp is also mandatory in the form of Unix Timestamp.

Pour exemple, when replacing old `0000_AAAABBBB.asc` with new `0001_CCCCDDDD.asc`, the clear-text signature file shall be named as `/root_keys/replacement_chain/0001_CCCCDDDD.txt.asc`.

### Delegation Declarations

Un delegation declaration consist of 2 parts:

- CA: Create Agent
- DR: Declare Root

Only when the 2 parts are complete shall the declaration be considered valid.

Delegation declarations are located in `/agent_keys/delegation_declarations`. Their filenames are in the form of `{Sequential|Hex-4}_{Fingerprint|Hex-8}_[CA,DR].txt.asc`.

Pour exemple, when creating un agent key `0000_11223344` for root key `0001_CCCCDDDD`, 2 files shall be present:

- `/agent_keys/delegation_declarations/0000_11223344_CA.txt.asc` (signed by CCCCDDDD)
- `/agent_keys/delegation_declarations/0000_11223344_DR.txt.asc` (signed by 11223344)

## Exchanging Encrypted Messages Over Insecure Layers

When Alice is composing un encrypted message to Bob, Alice shall add the currently valid root key and all currently valid agent keys of Bob to the list of recipients.

Bob shall recognize any digital signature from all agent keys of Alice, besides the currently valid root key of Alice.

## The Name

MNSY is ████████████ of ████████████████, namely, ███████████████████████████, █████████.

## Disclaimer

I do not grant, explicitly or implicitly, any person or organization, any authorization, at any degree, to further develop upon this protocol. I reserve, solely and fully, any right, if applicable, unless otherwise explicitly granted with consent, both written and digitally-signed, to any party.

I, in person or in corporation, may claim exclusive intellectual property rights of this protocol and/or any system thereof developed upon it.
