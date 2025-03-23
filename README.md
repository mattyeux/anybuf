# anybuf

This fork was made because I needed packed encoding and repeated message encoding.

[![anybuf on crates.io](https://img.shields.io/crates/v/anybuf.svg)](https://crates.io/crates/anybuf)
[![Docs](https://docs.rs/anybuf/badge.svg)](https://docs.rs/anybuf)

A minimal, zero dependency protobuf encoder and decoder to encode/decode anything.
It is designed to create the `value` bytes of a protobuf `Any`, hence the name.

Due to its low level design, anybuf allows you to do things wrong in many ways
and you should have a solid understanding of how protobuf encoding works in
general to better understand the API.

The crate anybuf is split in two major components:

- `anybuf::Anybuf` is a protobuf encoder
- `anybuf::Bufany` is a protobuf decoder

## Non goals

- protobuf 2 things
- Field sorting
- Groups support (deprecated, see <https://protobuf.dev/programming-guides/proto2/#groups>)

## Supported

- Varint fields (bool/uint32/uint64/sint32/sint64/int32/int64)
- Variable length fields (string/bytes)
- Nested messages: Just append an `Anybuf` instance
- Repeated (bool/uint32/uint64/sint32/sint64/int32/int64/string/bytes/messages)
- Packed encoding for repeated fields

## Not yet supported

- Fixed length types
- Maps support (but you can use the equivalent [encoding via repeated messages](https://protobuf.dev/programming-guides/encoding/#maps))

## How to use it

Encoding

```rust
use anybuf::Anybuf;

let data = Anybuf::new()
    .append_uint64(1, 4)        // field number 1 gets value 4
    .append_string(2, "hello")  // field number 2 gets a string
    .append_bytes(3, b"hello")  // field number 3 gets bytes
    .append_message(4, &Anybuf::new().append_bool(3, true)) // field 4 gets a message
    .append_repeated_uint64(5, &[23, 56, 192])              // field number 5 is a repeated uint64
    .into_vec();                // done

// data is now a serialized protobuf document
```

Decoding

```rust
use anybuf::Bufany;

let deserialized = Bufany::deserialize(&data).unwrap(); // data from above
let id = deserialized.uint64(1).unwrap(); // 4
let title = deserialized.string(2).unwrap(); // "hello"
```

## `no_std` support

Since version 0.5.0 there is a default `std` feature. If you remove that, the library is built with `no_std` support.
As the anybuf maintainers do not require `no_std` support this is provided at a best effort basis and might be broken.

```toml
[dependencies]
anybuf = { version = "0.5.0", default-features = false }
```
