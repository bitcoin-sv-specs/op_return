This segmented file protocol is intended to allow a file to be reconstituted from multiple parts located on the blockchain.

The idea of the segmented file protocol is to have a standard way to put files on the blockchain in OP_RETURN data, where the file is separated over multiple transactions or outputs.

A high level outline of a segmented file is as follows:

"sgfl" [filename] [segment-count] [data-reference]

In detail these are:

* "sgfl": A 4-byte prefix. In hex, it is 0x7367666c.
* [filename]: Pushdata containing a byte string. However, it is recommended that the filename be a utf8 character string. Furthermore, it is recommended that it be divided into name and extension like normal files: [name].[extension] . Operating systems are already used to dealing with filenames and parsing meaning from them, so we can reuse code from operating systems for this.
* [segment-count]: Pushdata containing a varint number, indicating the number of data-references that follow.
* [data-reference]: Pushdata containing a concatenated stream of, for each segment, a txid (32 bytes) and a varint tx output (variable length).

An example of a segmented file is as follows:

```
OP_RETURN 4 0x7367666c 9 0x626c6164652e6a7067 1 0x03 103 22cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b90222cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b9fd122722cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b9fd0002
```

This is the file 'blade.jpg' comprised of three parts, all in the same transaction, but in segments located in outputs 2, 10002 and 512.