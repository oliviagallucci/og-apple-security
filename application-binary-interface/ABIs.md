# Application Binary Interface (ABI)
An ABI is the low-level "contract" between compiled code and its execution environment. It defines:

- Calling conventions
  - how functions pass arguments (registers vs. stack), return values, and maintain stack alignment
- Data layout
  - sizes, alignments, and endianness of basic types and structures
- Object format & linking
  - binary file format, symbol mangling, relocation, and dynamic-linker behavior
- System-call interface
  - how applications invoke kernel services

## macOS ABIs

- Mach-O format
    - the native object/executable container for Intel and Apple Silicon, supporting "fat" (universal) binaries.
- Dynamic loader (dyld)
  - resolves and binds symbols at load-time or run-time, handling frameworks (_*.framework) and shared libraries (*.dylib).
- x86_64 (Darwin) ABI
  - Apple's variant of the System V AMD64 ABI—uses: 
    - RDI, RSI, RDX, RCX, R8, R9 for integer arguments; 
    - RAX for return; 
    - 16-byte stack alignment.
- ARM64 (Apple Silicon) ABI
  - follows AArch64 conventions—uses:
    - X0–X7 for arguments, 
    - X0 for return; 
    - 16-byte stack alignment; 
    - position-independent code via Mach-O slices.
- Universal binaries
  - single Mach-O files containing x86_64 and arm64 slices, allowing execution on Intel and Apple Silicon.
