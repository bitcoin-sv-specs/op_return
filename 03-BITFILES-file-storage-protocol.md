# BitFiles: Bitcoin SV File Storage Protocol

## Motivation:

With access to up to 100KB of space per transaction, the overhead of storing files on the Metanet is now much less than 1%.

To encourage the rapid development of compatible tools and utilities some standards and ground rules may be helpful ;-)

This specification is offered to kick off discussion.

If it survives and is encouraged by that process it aspires to lead at least to a working prototype for easily uploading, updating, listing, and retrieving files from Bitcoin SV.

## Objectives:

- Associate bitcoin address as an identifier for each file and to verify file ownership for updates.
- Handle files larger than current transaction limit.
- Handle multiple small files and related files in a single transaction.
- Support for content encodings.
- Support for content encryption and compression methods.
- Support for content replacement and incremental file updates (diffs).
- Support file metadata.
- Use PUSHDATA framing to facilitate block explorer access to key fields.
- Use Bitcom to register and manage protocol identifier.
- Resist attacks: Easily verify ownership and any attempted content updates.
- Minimal overhead and complexity.

## Description:

A bitcoin public address is used by this protocol as a file identifier.

To rewrite a file, submit a new transaction with new contents and the same public address. Alternatively, use an incremental update to change only a bit of the previous contents.

Large files and files larger than the current per transaction limit are accommodated by splitting files up into chunks. Chunks do not have to be the same size. The first chunk of a file is handled like one chunk files. Continuation chunks use their own protocol identifier and a simpler layout. The use of separate protocol identifiers allows for efficient retrieval of unique files and also of all the chunks relating to a single large file.

To better accommodate very small files and related files, multiple files can be included in a single transaction. For multiple very small files, this can reduce the transaction data overhead to effectively zero. For related files, first chunk sizes can be kept small if necessary.

The Bitcom meta protocol is specified here https://bitcom.bitdb.network.

The use of PUSHDATA for OP_RETURN data framing is specified here https://github.com/bitcoin-sv-specs/op_return/blob/master/01-PUSHDATA-data-element-framing.md.

## Specification:
	<first_chunk_protocol_identifier>
	[...this sequence can repeat...
		<file_identifier>
       <signature>
		<content_length>
		<metadata>
		<encoding_info>
		<encryption_info>
		<diff_info>
		<first_chunk_info>
		<content_bytes>
	]

	<continuation_chunk_protocol_identifier>
	[...this sequence does not repeat...
		<file_identifier>
       <signature>
		<continuation_chunk_info>
		<content_bytes>
	]

### <first_chunk_protocol_identifier>
A bitcoin address registered with the Bitcom protocol identifier registry.

Used only for the first chunk of each file.

### <continuation_chunk_protocol_identifier>
A bitcoin address registered with the Bitcom protocol identifier registry.

Used only for file continuation chunks.
 
### <file_identifier>
A file identifier is a bitcoin address. The associated private key is used to sign each update to this file and can serve as an encryption key.

Future protocol extensions may allow for excess miners fees paid from this address as compensation for long term access to this file.

### <signature>
This signature verifies that the transaction creator is in possession of the private key associated with the bitcoin address used as this file's identifier.

The process for creating the signature and verifying it is TBD.

###  <content_hash>
Still on the fence about the utility of a content hash.

###  <content_length>
Byte length of entire file across all chunks.

###  <metadata>
Varint followed by UTF8, JSON formatted metadata.

Metadata properties and value formats will be specified here ...

At least one diff mode will allow updating only metadata.

###  <encoding_info>
Varint byte length followed by data describing the encoding standard. Determines how raw (or decrypted) bytes are encoded.

A value of zero indicates no encoding specified.

###  <encryption_info>
varing byte length followed by data describing the encryption method. Determines how to decrypt encrypted content bytes to obtain raw content bytes.

A value of zero indicates no encryption.

###  <diff_info>
Varint byte length followed by a transaction ID and data describing the incremental difference algorithm to be used to apply these content bytes
to previous file version’s content bytes. The transaction ID determines the previous file version.

At least one diff mode will allow updating only metadata without having to respecify content, encryption, or encoding.

A value of zero indicates this is not an incremental file update. The content bytes completely replace any previous content bytes sharing the same file_identifier (bitcoin address).
Note that it may be useful to include a transaction ID even when completely replacing previous content to establish the files version history.

###  <first_chunk_info>
Varint specifying the maximum chunk index. When greater than zero, additional chunks with indices from one to this value are expected and the file’s contents
is reconstructed by arranging the chunks in order and applying the decryption, encoding, and diff data included with chunk zero.
When greater than zero it is followed by a varint specifying the number of content bytes in this chunk.

A value of zero indicates a single chunk file in which case the content_length has already specified the number of content_bytes.

###  <continuation_chunk_info>
Varint specifying this chunk’s non-zero index followed by a varint specifying the number of content bytes.

Zero is not a valid value.

###  <content_bytes>
Varint byte length followed by that many content bytes.

## Examples

### Simplest Case: A One-Chunk Unencrypted Binary File

	<first_chunk_pid>, <pub_key>, <content_length>, 0, 0, 0, 0, 0, <content_bytes>

### A Large, Three Chunk Unencrypted Binary File

	Transaction 1:
	<first_chunk_pid>, <pub_key>, <sig>, <content_length>, 0, 0, 0, 0, 2, <length0>, <content_bytes0>
	Transaction 2:
	<continuation_chunk_pid>, <pub_key>, <sig>, 1, <length1>, <content_bytes1>
	Transaction 3:
	<continuation_chunk_pid>, <pub_key>, <sig>, 2, <length2>, <content_bytes2>
