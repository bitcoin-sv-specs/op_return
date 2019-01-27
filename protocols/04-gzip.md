# GZIP protocol identifier

Protocol identifier: `0x677a6970`

## Specification

The protocol identfier is a `PUSHDATA` data element `0x04 0x677a6970` which is the 1 byte length prefix followed by the 4 byte identifier which translates to the ASCII string `gzip`.

The `gzip` protocol identifier is intended to markup another `OP_RETURN` based protocol by adding the simple rule:

If the `gzip` identifier is present the following `PUSHDATA` operation contains gzip encoded data.  The only exception is the very first `gzip` identifier which is used to designate that the gzip handler should parse all the `PUSHDATA` following according to the above rule.  To push a single gzipped data element the protocol identifier should be pushed twice followed by the gzipped data.

The second `PUSHDATA`following the gzip protocol identfier should also be interpreted as a protocol identfier.  In order to be interpreted according to the original protocol any data elements marked as gzipped should be decompressed and the gzip identifiers stripped out.

This protocol is stream friendly and can be implemented as a pipeline of stream handlers: `gzip_handler -> sub_handler`

### Examples 

##### Gzip a single data element: 

`<gzip> <gzip> <data_gz>`

Which should be returned from the gzip handler as:

`<data>`


##### Compressed FILE data but uncompressed FILEAME:

`'gzip' 'file' <filename> 'gzip' <data_gz>`

This should be passed to the gzip handler which should output:

`'file' <filename> <data>`

Which is then passed to the `file` protocol handler.

##### Compressed FILE and FILENAME:

`'gzip' 'file' 'gzip' <filename_gz> 'gzip' <data_gz>`

This should be passed to the gzip handler which should output:

`'file' <filename> <data>`

Which is then passed to the `file` protocol handler.

##### Compressed DATA but uncompressed TYPE:

`'gzip' 'data' <type> 'gzip' <data_gz>`

This should be passed to the gzip handler which should output:

`'data' <type> <data>`

Which is then passed to the `data` protocol handler.

