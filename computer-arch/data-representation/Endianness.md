# Endianness

Endianness specifies how multi‐byte values are laid out in memory:

- Big-endian - the most significant byte is stored at the lowest address.
- Little-endian - the least significant byte is stored at the lowest address.

Most systems are little endian. Memory technically stored backwards.

## Endianness on macOS

- Mach-O format
  - supports both endiannesses via distinct "magic" numbers in its header, allowing binaries for different architectures to coexist.
- PowerPC (pre-2006)
  - ran macOS in big-endian mode by default (though bi-endian variants existed).
- Intel (x86_64) & Apple Silicon (ARM64)
  - exclusively little-endian; 
  - both the CPU and dynamic loader (`dyld`) expect and produce little-endian data.
- Universal ("fat") binaries
  - may contain slices for multiple architectures, each with its own endianness flag in the Mach-O header.
