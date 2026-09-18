# cpp-base64

A fast, header-only Base64 encoder/decoder for C++20.

## Features

- **Header-only** - just drop `base64.hpp` into your project
- **Fast decoding** - uses a 256-entry lookup table for O(1) character validation
- **Correct padding handling** - properly encodes and decodes padded Base64
- **C++20** - uses `constexpr`, `noexcept`, and modern language features
- **No dependencies** - only requires the standard library (`<string>`, `<cstdint>`)
- **Namespace-isolated** - everything lives under `base64::`

## Requirements

- A C++20-compatible compiler (GCC 10+, Clang 10+, MSVC 19.29+)

## Installation

Copy `base64.hpp` into your project's include directory, or add it as a submodule:

```bash
git submodule add https://github.com/NepoOwen/cpp-base64.git
```

Then include it:

```cpp
#include "base64.hpp"
```

## Usage

### Encoding

```cpp
#include "base64.hpp"
#include <iostream>

int main() {
    std::string encoded = base64::encode("Hello, World!");
    std::cout << encoded << '\n'; // SGVsbG8sIFdvcmxkIQ==
}
```

### Decoding

```cpp
#include "base64.hpp"
#include <iostream>

int main() {
    std::string decoded = base64::decode("SGVsbG8sIFdvcmxkIQ==");
    std::cout << decoded << '\n'; // Hello, World!
}
```

### Round-trip

```cpp
std::string original = "The quick brown fox jumps over the lazy dog";
std::string encoded  = base64::encode(original);
std::string decoded  = base64::decode(encoded);

assert(original == decoded);
```

## API

### `std::string base64::encode(const std::string& input)`

Encodes the given byte string into a standard Base64 string (RFC 4648). Output is padded with `=` when the input length is not a multiple of 3.

- **Parameters:** `input` - the raw bytes to encode
- **Returns:** the Base64-encoded string

### `std::string base64::decode(const std::string& input)`

Decodes a standard Base64 string back into raw bytes. Stops at the first `=` padding character. Whitespace and unrecognized characters are silently skipped. Returns as much data as could be decoded.

- **Parameters:** `input` - the Base64 string to decode
- **Returns:** the decoded byte string

## Implementation Notes

- **Decode table (`kDecodeTable`):** A `constexpr` 256-entry table maps every byte to its 6-bit Base64 value. Special sentinels:
  - `0xFE` - padding character `=`
  - `0xFF` - invalid / whitespace character
  - `0x00`–`0x3F` - valid 6-bit values
- **Block decoding:** Four input characters are packed into a single `uint32_t` and split into three output bytes, minimizing branching in the hot path.
- **Tail handling:** After the block loop, remaining characters are decoded bit-by-bit to correctly handle truncated or padded input.
- **Encoding:** Processes 3 input bytes at a time into 4 output characters; the final partial block is padded with `=`.

## Performance

The decode path uses a single table lookup per character with a fast combined validity check (`(a | b | c | d) >= 0xFE`), avoiding per-character branches. The encode path is a straightforward 3→4 byte expansion with no conditionals inside the main loop.

## License

This project is provided as-is. See the repository for license details.

## Author

**NepoOwen** - [https://github.com/NepoOwen](https://github.com/NepoOwen)

## Contributing

Issues and pull requests are welcome. Please keep changes header-only and dependency-free.
