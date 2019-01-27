# data in OP_RETURN

Protocol identifier: `0x64617461`

## Specification

The protocol identfier is a `PUSHDATA` data element `0x04 0x64617461` which is the 1 byte length prefix followed by the 4 byte identifier which translates to the ASCII string `data`.

This should be followed by a two further `PUSHDATA` data elements: `<type> <data>`

### Type

It is recommended that the `type` field contain a utf8 character string describing the data type. A list of well recognised types may be the subject of a later specification. However allowed valuers are not currently part of this specification.

Example types might include: `bin`, `json`, `jpeg`, `xml`, `html`, `pdf`, `txt`.  IANA content types may be supported as well.


### Data 
Any length of data in any format. However, it is recommended that the data comply with the format specified in the `type` field.

### Example

An example of a data item is as follows:

```
OP_RETURN 0x04 0x64617461 0x04 6a736f6e 0x0f 0x7b226b6579223a2276616c7565227d
```

This is data of type 'json' with a data value of: "{"key":"value"}"


