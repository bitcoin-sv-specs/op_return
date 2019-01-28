# data in OP_RETURN

Protocol identifier: `0x6d657461`

## Description

The `meta` protocol is a mechanism of adding meta data to any other protocol by encapsulating it.

## Specification

The protocol identfier is a `PUSHDATA` data element `0x04 0x6d657461` which is the 1 byte length prefix followed by the 4 byte identifier which translates to the ASCII string `meta`.

This should be followed by a further `PUSHDATA` data element: `<metadate>` which is a a JSON encoded set of the key value pairs.  The key value pairs remain unspecified.

This should be followed by another protocol id and it's subsequent data elemts in their entirety.

### Example

We may for example wish to add authorship and encoding data to a text data item.  Using the `data` protocol we should do it like this:

`'meta' '{"author": "alice", "encoding":"utf8"}' 'data' 'txt' 'I am a fish'`

Which would encode to:
`0x04 0x6d657461 0x26 0x7b22617574686f72223a2022616c696365222c2022656e636f64696e67223a2275746638227d 0x04 0x64617461 0x03 747874 0x0b 0x4920616d20612066697368`


