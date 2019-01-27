The idea of the file protocol is to have a standard way to put files on the blockchain in OP_RETURN data.

A high level outline of a file is as follows:

"file" [filename] [filedata]

Each part is preceeded by the appropriate pushdata command. filename and filedata can be any length.

In detail these are:

* "file": A 4-byte prefix. In hex, it is 0x66696c65.
* [filename]: Any byte string. However, it is recommended that the filaname be a utf8 character string. Furthermore, it is recommended that it be divided into name and extension like normal files: [name].[extension] . Operating systems are already used to dealing with filenames and parsing meaning from them, so we can reuse code from operating systems for this.
* [filedata]: Any length of data in any format. However, it is recommended that producers format the file correctly according to the extension. For instance, if your filename is mydocument.pdf then the filedata should be a properly valid PDF file.

This structure for files on the blockchain is extremely simple and allows us to reuse the same code for understanding files that already existing in operating systems. Block explorers can start displaying all ordinary filetypes right away. Files that are not valid won't be understood by generic block explorers, but may be understood by proprietary applications.

An example of a file is as follows:

```
OP_RETURN 4 0x66696c65 18 68656c6c6f2e747874 6 68656c6c6f0a
```

This is the file hello.txt consisting of the word "hello" followed by a newline.
