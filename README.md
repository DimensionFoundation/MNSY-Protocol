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
- **Separation of Fields**: The separation between fields is exactly 2 `\n`, because base-64 encoded contents in the value parts may contain `\n`.
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

X-Application-ID: 2708d651-7ad6-489c-a634-e2d57df21121

Application-Access-Level: 25

Application-User-Id: 18728374567

Application-Device-Session-Id: 0d79d73e-d4f0-408a-b12b-2246221d2ad8

XXXXXXX-END-MNSY-MESSAGE-PAYLOAD-CLEAR-XXXXXXX
-----BEGIN PGP SIGNATURE-----

iQIzBAEBCgAdFiEEaAYa8MGBOITvEbUKKv3DqQle2JIFAlxQU28ACgkQKv3DqQle
2JKopw//Zt5Pt3MPFxi5vHqURkB3PfTGqL/l/t+7AT19isRknmtFWrgzsZCjXNKQ
qlngdUsUhnDUltO8Dgf4cfABeEsLZs6VCMEKDAJUHtwnM1fDiwQ2p6RK/MUlN6wx
NpDKcFrjKZ4Brr1ezl9EZ/kmJcParLuU2yq4KW4vEYGndwmm6fk03ewoAKREmLzY
VV9BCCYm0tWtMtgMZO6LnDeyzeuw4DxpXcL+S9I4xIxnOztFr9P549HRg+0rytxg
nJ2cYfQhpvzbAhst0ltKnW9U6VlbjFYPVoEzlzS4ZtIRR6JkzAo/19mFng+5sddu
jdXAZ4H8j9KMs1WlqMhypOsRfZh/eTv82msMUGYpIeaxevG8p4MO98VfebsxOVBi
/gGBMKpznB+w9uNCy5sBjhC/860TK0//L3qCpy9UtZrc9mHJHl/HxmihfVW3Tdcg
fU9bvFUFkwmLBpgeWgLkimdPjwKsf/KqG4Jp4k0e10cgUYGJvLBHQpCS/jWAUFHy
MqVvVCOhuWYmVrhKXcTsk/b8H9yi21nNkPpI/FXeZzgz0o6G6mS8r8+4mgvgmmA7
UOGU1NaZniYByDWPm1ZStM74mbK/nQE4ndI+xcc3P5BsLVoBfbJ2TvgZq3ZYwaGS
gtjYHJMbPOneGrrUgBAs2ZXJldxDvyrwUuuLRfSwkKSY3BN7oDM=
=bMpl
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

### Root-Key-Replacement

### Create-Agent-For-Root-Key

### Declare-Root-Of-Agent-Key

### Authorize-Instance-Key

### Key-Transfer-Chunk

## Query Model over HTTPS

Query services are recommended to serve over HTTPS in order to take advantage of public services like GitHub Pages and CloudFlare.

Each query service shall have un query string template to generate un URI for the desired information, which may look like:

```
https://mnsy-idqs.neruthes.0xcc.club/db/{T}/{Q}
https://mnsy-idqs.neruthes.0xcc.club/query.do?v={T}&q={Q}
https://github.com/neruthes/mnsy-idqs/raw/master/db/{T}/{Q}
```

When ever there is a query request, the client shall replace `{Q}` with the encoded query string body, and `{T}` with the query type index.

### Query String Formation Overview

| Type  | Query String                         | Result Format
| ----- | ------------------------------------ | ------------------------------------
| 0-0   | {Fingerprint\|hex-40}                | Public key
| 0-1   | {Fingerprint\|hex-40}-cadr           | Create-Agent & Declare-Root
| 0-2   | {Fingerprint\|hex-40}-ca             | Create-Agent
| 0-3   | {Fingerprint\|hex-40}-dr             | Declare-Root
| 1-0   | {Fingerprint\|hex-40}-meta           | Public key metadata
| 1-1   | {Fingerprint\|hex-40}-chains         | Related replacement chains
| 1-2   | chain-{SHA256\|hex}                  | Replacement chain
| 2-0   | {Email Address}(-[R,A])?             | Matching fingerprints
| 2-0   | {Name}(-[R,A])?                      | Matching fingerprints
| 2-0   | {Fingerprint\|hex-8~39}(-[R,A])?     | Matching fingerprints

Notes:

- All inputs shall be interpreted case-insensitively.
- `[R,A]`: Must choose one between R (Root) and A (Agent) to specify role in the hierarchy.
- `(...)?`: This part can be omitted.

### Public key metadata

```
{
    "fingerprint": "68061AF0C1813884EF11B50A2AFDC3A9095ED892",
    "length": 4096,
    "validity": "valid",
    "name": "Neruthes",
    "emails": [
        "i@neruthes.xyz",
        "i@neruthes.com",
        "i@joyneop.xyz",
        "i@joyneop.com"
    ],
    "mnsy": {
        "mnsy": true,
        "class": "agent",
        "root": "80690EBD599119050EF943AF037F2FB3D5E2F043"
    },
    "public_followers": [
        "5AFDB16B89805133F450688BDA580D1D5F5CC7AD",
        "F42C2D1DC0F597A4223C2FA629D014EA006B70CB"
    ]
}
```

### Related replacement chains

```
{
    "query_type": "1-1",
    "query_input": "80690EBD599119050EF943AF037F2FB3D5E2F043",
    "output_type": "1-1-0",
    "//": "Index only",
    "output": [
        "5AFDB16B89805133F450688BDA580D1D5F5CC7AD"
    ]
}
```

```
{
    "query_type": "1-1",
    "query_input": "80690EBD599119050EF943AF037F2FB3D5E2F043",
    "output_type": "1-1-1",
    "//": "With each chain given",
    "output": [
        {
            "chain_sha256": "884e153884d693c0ca3d10e30d975770af2204c019ef351d8a2f800f42fa3681",
            "chain": [
                "8F45D72357EEB6C556662C8D68A0B4B6EEF9762A",
                "80690EBD599119050EF943AF037F2FB3D5E2F043"
            ]
        }
    ]
}
```

### Replacement chain

```
{
    "query_type": "1-2",
    "query_input": "Neruthes",
    "output": {
        "chain_sha256": "884e153884d693c0ca3d10e30d975770af2204c019ef351d8a2f800f42fa3681",
        "chain": [
            "8F45D72357EEB6C556662C8D68A0B4B6EEF9762A",
            "80690EBD599119050EF943AF037F2FB3D5E2F043"
        ]
    }
}
```

### Matching fingerprints

```
{
    "query_type": "2-0",
    "query_input": "Neruthes",
    "output": [
        "80690EBD599119050EF943AF037F2FB3D5E2F043",
        "8F45D72357EEB6C556662C8D68A0B4B6EEF9762A"
    ]
}
```

Query type indexes are always decimal integers.

## Exchanging Encrypted Messages Over Insecure Layers

When Alice is composing un encrypted message to Bob, Alice shall add the currently valid root key and all currently valid agent keys of Bob to the list of recipients.

Bob shall recognize any digital signature from all agent keys of Alice, besides the currently valid root key of Alice.

## The Name

MNSY is ████████████ of ████████████████, namely, ███████████████████████████, █████████.

## Disclaimer

I do not grant, explicitly or implicitly, any person or organization, any authorization, at any degree, to further develop upon this protocol. I reserve, solely and fully, any right, if applicable, unless otherwise explicitly granted with consent, both written and digitally-signed, to any party.

I, in person or in corporation, may claim exclusive intellectual property rights of this protocol and/or any system thereof developed upon it.
