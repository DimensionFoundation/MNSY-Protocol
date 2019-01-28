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
- **Prefixes**: Protocol-level constant fields start with `XXX`, and subprotocol-level ones `X`. Prefixes that consist of purely `X` are reserved by MNSY Protocol; developers may use other names for their fields. Un prefix is the result of `^[A-Z]-`; if the regexp match nothing, then the key of the field have no prefix.

### PGP Integration

## Subprotocols

| XXX-Protocol-ID                      | XXX-Protocol-Name
| ------------------------------------ | -----------------
| 00000000-0000-0000-0000-000000000000 | Root-Key-Replacement
| 00000000-0000-0000-0001-000000000000 | Create-Agent-For-Root-Key
| 00000000-0000-0000-0001-000000000001 | Declare-Root-Of-Agent-Key
| 00000000-0000-0000-0002-000000000000 | Authorize-Instance-Key
| 00000000-0000-0000-0003-000000000000 | Key-Transfer-Chunk

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
