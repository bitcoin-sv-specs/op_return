# Extensible 4-byte prefix guideline for OP_RETURN

**Abstract:** As an _optional guideline_, we recommend that all present and future protocols to be implemented on Bitcoin SV implement a 4-byte prefix scheme, referred to as protocol identifiers, whenever they use `OP_RETURN`. This scheme will simplify interoperability between protocols, facilitate selective pruning of the blockchain and get built-in support from low-level infrastructure components. The proposal also provides a basic early stage process to let the ecosystem participants claim prefixes for their own protocols.

_In the following, Bitcoin always refers to Bitcoin SV._


## Overview

The Bitcoin ecosystem has gained renewed interest for overlay protocols built on top of the blockchain, leveraging the growing capacity of the `OP_RETURN` opcode of Bitcoin to embed arbitrary data in Bitcoin transaction. In order simplify parsing of structured data carried in `OP_RETURN` a simple message to identify data relevant to a use case or ignore data that is irrelevant a recommended method id prefixing messages carried through `OP_RETURN` with a protocol identifier to enable simple sorting of messages according to their respective protocols. 

An agreed unifying scheme will help to avoid collisions between protocols. While none of those collisions endanger Bitcoin itself, they will significantly and needlessly complicate the design of the software intended to operate those overlay protocols.

## Guideline

A 4 byte data element containing the protocol should be the first element of data following the `OP_RETURN` op code. The data element should be length prefixed as described in the [PUSHDATA framing specfication](01-PUSHDATA-data-element-framing.md).  That is the byte 0x04 followed by the 4 bytes of the protocol identifier.

If the left most byte of the protocol identifier is 0xff it is a marker to indicate the protocol identifier is extended.  And additional PUSHDATA data element immediately after the first one MUST follow.  This data element can be of arbitrary length and the data should be appended to the last 3 bytes of the first data element to form the full protocol identifier.

## Protocol identifier registry

As a courtesy to the community, we recommend to either submit a ticket or a pull request to the Git repository at https://github.com/bitcoin-sv-specs/op_return to claim your prefix with:

* A display name for the protocol
* An author (or list of authors)
* A URL pointing to the specification of the protocol
* A Bitcoin address (to avoid ambiguity and to later modify names / authors / URL).

This last step is _not_ a requirement. However, if you don’t try to make your prefix known to the community at large, be aware that it leaves you open to collisions with a useful protocol which just happens to have a lot more traction than yours.

See also _Annex: file format of /protocol_ids.csv

## Annex: file format of /protocol_ids.csv

In order to make known protocols easily available to the community at large, a simple file format is proposed to gather the protocol prefixes. 

This file `protocol_ids.csv` should be seen as an early stage effort to help various protocols gain traction within the Bitcoin community. If the number of active protocols becomes greater than a few hundred, we expect that the file `protocol_ids.csv` will be superseded with an approach more scalable than having a flat text file holding all known protocols.

The URL for the file is expected to be:

https://github.com/bitcoin-sv-specs/op_return/blob/master/etc/protocol_ids.csv 

The file is encoded in CSV as per [RFC 4180](https://tools.ietf.org/html/rfc4180) with the following options:

* UTF-8 encoding
* Unix line ending (\n)
* Comma delimiter
* Optional quote escaping for strings
* Quote escaped strings cannot contain newline (\n), returns (\r) or quotes (“)
* First line is the column headers

Then the columns themselves:

* `Prefix`: hexadecimal encoded (aka 0x01234567)
* `DisplayName`: string
* `Authors`: string
* `BitcoinAddress`: a valid Bitcoin address
* `SpecificationUrl`: string
* `TxidRedirectUrl` (optional): string (contains `{txid}`)

Each line should not be longer than 1KB (1024 bytes) in total.

The lines should be sorted in increasing order against their prefix.

The field `BitcoinAddress` is intended to help resolve any conflicting claims in the event where such claims were to arise.

The field `TxidRedirectUrl` is intended to help blockchain explorers making protocols more discoverable. For any transaction associated to the protocol - as identified through its prefix - a redirecting link can be inserted. The field `TxidRedirectUrl` should contain the substring `{txid}` to be replaced by the transaction identifier encoded in hexadecimal (64 characters). The landing page of the redirect is expected to be a human-readable version of the transaction aligned with the semantic of the protocol.
