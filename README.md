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

MNSY is the initialism of my name `Neruthes` and the names of core contributors, namely, `Mindey`, `Suji Yan`, and `Yisi Liu`, as sorted in Unicode sequential order.

## Disclaimer

I do not grant, explicitly or implicitly, any person or organization any authorization, at any degree, to further develop upon this protocol. I reserve, solely and fully, any right, if applicable, unless otherwise explicitly granted with consent, both written and digitally-signed, to any party.

I, in person or in corporation, may claim exclusive intellectual property rights of this protocol and/or any system thereof developed.
