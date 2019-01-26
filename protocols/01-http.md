# HTTP in OP_RETURN

Protocol identifier: `0x68747470`

## Specification

The protocol identfier is a `PUSHDATA` data element `0x04 0x68747470` which is the 1 byte length prefix followed by the 4 byte identifier which translates to the ASCII string `http`.

This should be followed by a two further `PUSHDATA` data elements: `<headers> <body>`

### Headers 

Should contain a set of valid HTTP headers encoded as JSON.  Repeated keys are support by the value being encoded as a JSON array of elements.

headers are typically used to specfiy content type and encoding.

### Body 
Should contain the document body as specified by the content-type in the header.


