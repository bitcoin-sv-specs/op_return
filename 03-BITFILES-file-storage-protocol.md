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
- Support for media type and content encodings.
- Support for content encryption and compression methods.
- Support for content replacement and incremental file updates (diffs).
- Use PUSHDATA parameter framing including null parameters.
- Use Bitcom to register and manage protocol identifiers.
- Resist attacks: Easily verify ownership and any attempted content updates.
- Minimal overhead and complexity ;-)

## Description:

A bitcoin public address is used by this protocol as a file identifier.

To rewrite a file, submit a new transaction with new contents and the same public address. Alternatively, use an incremental update to change only a bit of the previous contents.

Large files and files larger than the current per transaction limit are accommodated by splitting files up into chunks. Chunks do not have to be the same size. The first chunk of a file is handled like one chunk files. Continuation chunks use their own protocol identifier and a simpler layout. The use of separate protocol identifiers allows for efficient retrieval of unique files and also of all the chunks relating to a single large file.

To better accommodate very small files and related files, multiple files can be included in a single transaction. For multiple very small files, this can reduce the transaction data overhead to effectively zero. For related files, first chunk sizes can be kept small if necessary.

The Bitcom meta protocol is specified here https://bitcom.bitdb.network.

The use of PUSHDATA for OP_RETURN data framing is specified here https://github.com/bitcoin-sv-specs/op_return/blob/master/01-PUSHDATA-data-element-framing.md.

## Metadata:

Metadata associated with a particular file can be stored and updated using a separate file meta data protocol...

## Specification:
	<first_chunk_protocol_identifier>
	[...this sequence can repeat...
		<file_identifier>
       <signature>
		<media_type_info>
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

The process for creating the signature and verifying it is...

###  <media_type_info>
Media type of content as listed at https://www.iana.org/assignments/media-types/media-types.xhtml.

###  <encoding_info>
Encoding of content (post decryption, decompression if any) as listed at https://www.iana.org/assignments/character-sets/character-sets.xhtml.

###  <encryption_info>
Determines what decryption or decompression algorithms to apply to content bytes.

Initial options may include:
- gzip
- pk
 
Multiple options can be combined with pipe separators.

A value of "pk" indicates the content has been encrypted using the file_identifier's private key.

A value of "gzip" indicates the content was compressed using the gzip algorithm.

A value of zero, OP_0, indicates no encryption or compression.

###  <diff_info>
Transaction ID followed by a parameter describing the incremental difference algorithm to be used to apply these content bytes
to previous file version’s content bytes. The transaction ID determines the previous file version.

A value of zero, OP_0, for the transaction ID indicates the start of a new file revision history.

A value of zero, OP_0, for the difference algorithm indicates this is not an incremental file update.
The content bytes completely replace any previous content bytes sharing the same file_identifier (bitcoin address).

###  <first_chunk_info>
Maximum chunk index. When greater than zero, additional chunks with indices from one to this value are expected and the file’s contents
is reconstructed by arranging the chunks in order and applying the decryption, encoding, and diff data included with chunk zero.

A value of zero, OP_0, indicates a single chunk file.

###  <continuation_chunk_info>
This chunk’s non-zero index, from 1 to the maximum specified with first chunk.

Zero is not a valid value.

###  <content_bytes>
Content bytes in this chunk. For single chunk files, the PUSHDATA specifies the total file contents length.

## Examples

### Simplest Case: A One-Chunk Unencrypted Binary Image File

	<first_chunk_pid> <pub_key> <sig> image/png binary OP_0 OP_0 OP_0 OP_0 <content_bytes>

### A Large, Three Chunk Unencrypted Binary Image File

	Transaction 1:
	<first_chunk_pid> <pub_key> <sig> image/png binary OP_0 OP_0 OP_0 2 <content_bytes0>
	Transaction 2:
	<continuation_chunk_pid> <pub_key> <sig> 1 <content_bytes1>
	Transaction 3:
	<continuation_chunk_pid> <pub_key> <sig> 2 <content_bytes2>
