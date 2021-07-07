# Segmented file protocol

The idea of the segmented file protocol is to have a standard way to put files on the blockchain in OP_RETURN data, where the file is separated over multiple transactions or outputs.

## Protocol

A high level outline of a segmented file is as follows:

"sgfl" [filename] [data-reference]

In detail these are:

* "sgfl": A 4-byte prefix. In hex, it is 0x7367666c.
* [filename]: Pushdata containing a byte string. However, it is recommended that the filename be a utf8 character string. Furthermore, it is recommended that it be divided into name and extension like normal files: [name].[extension] . Operating systems are already used to dealing with filenames and parsing meaning from them, so we can reuse code from operating systems for this.
* [data-reference]: Pushdata containing a concatenated stream of, for each segment, a txid (32 bytes) and a varint tx output (variable length).

An example of a segmented file is as follows:

```
OP_RETURN 4 0x7367666c 9 0x626c6164652e6a7067 103 22cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b90222cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b9fd122722cd5757d5194daa014944128cbf86261a5f42e78c2f4e541b6b7774297e74b9fd0002
```

This is the file 'blade.jpg' comprised of three parts, all in the same transaction, but in segments located in outputs 2, 10002 and 512.

## Segment contents

These are the ways segments are processed:

* If the segment is recognised as being defined by this segmented file protocol, the reconstructed data is used as the segment data.
* If the segment is recognised as being defined by the file protocol, the embedded file data is used as the segment data.
* Otherwise the data within the first pushdata in the transaction output references as the segment, is used as the segment data.

## On disk representation

The complete OP_RETURN contents (excluding the leading OP_RETURN opcode) is stored in an on-disk file, perhaps as '[filename].bitcoin'.  It can be identified and processed using the same processing that identifies and processes the contents of an OP_RETURN.