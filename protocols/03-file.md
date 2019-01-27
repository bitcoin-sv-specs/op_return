file protocol
=============

The idea of the file protocol is to have a standard way to put files on the blockchain in OP_RETURN data.

A high level outline of a file is as follows:

"file" [filename] [filedata]

Each part is preceeded by the appropriate pushdata command. filename and filedata can be any length.

In detail these are:

* "file": A 4-byte prefix. In hex, it is 0x66696c65.
* [filename]: Any byte string. However, it is recommended that the filaname be a utf8 character string. Furthermore, it is recommended that it be divided into name and extension like normal files: [name].[extension] . Operating systems are already used to dealing with filenames and parsing meaning from them, so we can reuse code from operating systems for this.
* [filedata]: Any length of data in any format. However, it is recommended that producers format the file correctly according to the extension. For instance, if your filename is mydocument.pdf then the filedata should be a properly valid PDF file.

This structure for files on the blockchain is extremely simple and allows us to reuse the same code for understanding files that already exist in operating systems. Block explorers can start displaying all ordinary filetypes right away. Files that are not valid won't be understood by generic block explorers, but may be understood by proprietary applications.

An example of a file is as follows:

```
OP_RETURN 4 0x66696c65 18 0x68656c6c6f2e747874 6 0x68656c6c6f0a
```

This is the file hello.txt consisting of the word "hello" followed by a newline.

Another example is a ".bitcoin" file format that contains a list of txids. Large files can be reconstructed by looking at the files inside and putting them back together in order. If a tx has more than one OP_RETURN, use the first one. A .bitcoin file can contain other .bitcoin files if necessary to store huge amounts of data.

FAQ
===

**What if the tx has more than one OP_RETURN?** Each OP_RETURN can be a different file.

**How do I store meta data?** Meta data is not included in this spec.

**What if I want to do a bunch of other things not specificed here?** Create a new OP_RETURN protocol.
