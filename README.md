(🚫)
======
Note:  This is a stripped down version of Nostr NIP-01: https://github.com/nostr-protocol/nips/blob/master/01.md

Description
-------------------------------

🚫 is a data schema that has no defined protocol, no identity, no references, no tags, no video attachments, and no image attachments.  The data format is compatible with Nostr. It is used for simple collaboration without servers or administrators.

Why 🚫?
-------------------------------

By stripping away the protocol, it is possible to use MQTT, Syslog, Nostr, or even ATproto to share messages. The primary feature of 🚫 is using a standard integrity signature.  This seems counter to having no identity; however, there are no mechanisms for friendly identities provided.  All that is known is that the messages come from an agent that is trusted to control a private key to sign a message.  The author's use case is solving tactical problems at time of crisis via collaborative knowledge graphs over MQTT using single page web apps and local storage; however, many other applications are possible.

## Events and signatures

Each user has a keypair. Signatures, public key, and encodings are done according to the [Schnorr signatures standard for the curve `secp256k1`](https://bips.xyz/340).

The only object type that exists is the `event`, which has the following format on the wire:

```yaml
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized event data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the event creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": 1,
  "tags": [],
  "content": <arbitrary string>,
  "sig": <64-bytes lowercase hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
```

To obtain the `event.id`, we `sha256` the serialized event. The serialization is done over the UTF-8 JSON-serialized string (which is described below) of the following structure:

```
[
  0,
  <pubkey, as a lowercase hex string>,
  <created_at, as a number>,
  <kind, as a number>,
  <tags, as an empty array>,
  <content, as a string>
]
```

To prevent implementation differences from creating a different event ID for the same event, the following rules MUST be followed while serializing:
- UTF-8 should be used for encoding.
- Whitespace, line breaks or other unnecessary formatting should not be included in the output JSON.
- The following characters in the content field must be escaped as shown, and all other characters must be included verbatim:
  - A line break (`0x0A`), use `\n`
  - A double quote (`0x22`), use `\"`
  - A backslash (`0x5C`), use `\\`
  - A carriage return (`0x0D`), use `\r`
  - A tab character (`0x09`), use `\t`
  - A backspace, (`0x08`), use `\b`
  - A form feed, (`0x0C`), use `\f`

### Tags

While the tags array exists, it is empty.

### Kinds

A kind value of 1 is always used.  This means that the content, per [Nostr NIP-10](https://github.com/nostr-protocol/nips/blob/master/10.md), is a human-readable text note, and markup languages such as markdown and HTML SHOULD NOT be used. 
 
