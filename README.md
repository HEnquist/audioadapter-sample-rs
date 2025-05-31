# audioadapter-sample

The `audioadapter-sample` crate helps with handling audio samples in various formats.
It is part of the `audioadapter` family.


## Introduction
Audio data is stored and exchanged in various numerical formats,
for example 16 or 24 bit integers, or 32-bit floating point.
When data is exchanged via files or APIs,
the sample values have to be stored in some binary format.
This crate is intended to help with both conversions between the various numerical formats,
as well as handling reading and writing these samples as raw bytes.

## Sample formats
This crate supports signed and unsigned integer samples with 8, 16, 24, 32 and 64 bits.
Floating point samples are also supported, with 32 and 64 bits.
For all formats, both little-endian and big-endian representations are supported.

When converting between integers and floating point, the value range of the integer
is mapped to the floating point range of -1.0 to +1.0.
If a floating point value outside this range is converted to integer,
the value will be clamped at the miminum or maximum value of the integer.

## This crate
The main functionality of this crate is provided by two main traits:
- [sample::BytesSample] - conversions between raw bytes and the (nearest) corresponding numeric type.
- [sample::RawSample] - conversions between different numeric types.

In addition to the traits, it also defines a number of wrappers for bytes.
For example, four bytes might be a signed or unsigned 32-bit integer, a 32-bit float, or
a 24-bit integer with a padding byte.
By using the appropriate wrapper, for example [sample::I32LE], it becomes clear that
these bytes store a signed 32-bit integer in little-endian byte order.

## Integer formats with different bit depths
Some integer sample formats match a standard integer type such as [i16] or [i32].
However there is no direct match for 24-bit samples.
24-bit samples are also commonly stored in two different ways:
as either 3 bytes per sample, or 4 bytes per sample (with an extra byte of padding).

This crate provides ways to convert between all the common integer sample formats
and the standard integer types.
For 24-bit samples, the next larger type, [i32] or [u32], is used,
such that the conversion can be done without loss of precision.

### Example: read 24-bit integers from raw bytes
```rust
use audioadapter_sample::sample::I24LE;
use audioadapter_sample::sample::BytesSample;

// Make a vector with some dummy data.
// 9 bytes corresponding to 3 samples.
let bytes = vec![1, 2, 3, 4, 5, 6, 7, 8, 9];

for sample in bytes.chunks_exact(3) {
  let new_value: i32 = I24LE::<3>::from_slice(sample).to_number();
  println!("{}", new_value);
}
```

## Converting between numerical formats
A very common sample format is 16-bit signed integer.
But when doing any kind of processing of audio data, such as filtering or mixing,
it is often desirable to work with floating point values.
An application may therefore need to convert input data from 16-bit integers
to floating point, perform the needed processing,
and convert back to 16-bit integers before returning the data.

### Example, converting [i16] to and from [f32]
```rust
use audioadapter_sample::sample::RawSample;

// make a vector with some dummy data.
let indata: Vec<i16> = vec![1, 2, 3, 4];
let mut outdata: Vec<i16> = Vec::new();

for sample in indata.iter() {
  // convert to f32
  let float_sample: f32 = sample.to_scaled_float();
  // do some processing
  let processed_sample = 0.99 * float_sample;
  // convert back to 16-bit integer
  let processed_int = i16::from_scaled_float(processed_sample);
  if processed_int.clipped {
    println!("The value was clipped during conversion to integer");
  }
  outdata.push(processed_int.value);
}
```

### Example, converting 16-bit integers from raw bytes to [f32]
```rust
use audioadapter_sample::sample::I16LE;
use audioadapter_sample::sample::BytesSample;
use audioadapter_sample::sample::RawSample;

// Make a vector with some dummy data.
// 6 bytes corresponding to 3 samples.
let bytes = vec![1, 2, 3, 4, 5, 6];

for sample in bytes.chunks_exact(2) {
  let new_value: f32 = I16LE::from_slice(sample).to_scaled_float();
  println!("{}", new_value);
}
```


## Reading and writing samples from types implementing `Read` and `Write`
The [std::io::Read] and [std::io::Write] traits are useful for reading
and writing raw bytes to and from for example files.
The [readwrite] module extends these traits by providing methods for reading and writing samples,
with on-the-fly conversion between bytes and the numerical values.
This functionality depends on the standard library ans is gated by the `std` Cargo feature.

Example
```rust
# #[cfg(feature = "std")]
# {
use audioadapter_sample::sample::I16LE;
use audioadapter_sample::readwrite::ReadSamples;

// make a vector with some dummy data.
let data: Vec<u8> = vec![1, 2, 3, 4];
// slices implement Read.
let mut slice = &data[..];
// read the first value as 16 bit integer, convert to f32.
let float_value = slice.read_converted::<I16LE, f32>();
# }
```

## Compatibility with the [audio](https://crates.io/crates/audio) crate
The wrappers for bytes implement the [audio_core::Sample] trait from the [audio](https://crates.io/crates/audio) crate.
This is controlled by the `audio` feature which is enabled by default.

## Cargo features
This crate has the following features:
 - `std` enables the standard library.
 - `audio` enables `audio` crate compatibility.

Both features are enabled by default.

## License: MIT
