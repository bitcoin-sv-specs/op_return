Thanks to Ryan X Charles for the [original spec](https://github.com/bitcoin-sv-specs/op_return/blob/5f051997061ea45873e51a04b494387ad40df05e/protocols/03-file.md)

# Files in OP_RETURN

Protocol identifier: `0x66696c65`

## Specification

The protocol identfier is a `PUSHDATA` data element `0x04 0x66696c65` which is the 1 byte length prefix followed by the 4 byte identifier which translates to the ASCII string `file`.

This should be followed by a two further `PUSHDATA` data elements: `<filename> <data>`

### Filename

Contains any data element. However, it is recommended that the filaname be a utf8 character string. Furthermore, it is recommended that it be divided into name and extension like normal files: `<name>.<extension>` . Operating systems are already used to dealing with filenames and parsing meaning from them, so we can reuse code from operating systems for this.


### Data 
Any length of data in any format. However, it is recommended that producers format the file correctly according to the extension. For instance, if your filename is mydocument.pdf then the filedata should be a properly valid PDF file.

### Example

An example of a file is as follows:

```
OP_RETURN 0x04 0x66696c65 0x09 0x68656c6c6f2e747874 0x06 0x68656c6c6f0a
```

This is the file hello.txt consisting of the word "hello" followed by a newline.


