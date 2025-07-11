# Disassembly

Disassembly converts a compiled binary's machine code into assembly instructions. It preserves low-level detail and is essential for instruction-level analysis.... Translating raw opcodes into assembly mnemonics, registers, and constants.

Uses: 

* Vulnerability research (ROP gadgets, calling conventions)
* exploit dev
* Bit-level and flag behavior analysis

## Workflow (?)

1. Input: Binary executable or library
2. Engine: Opcode decoder + CPU architecture spec
3. Output: Linear or graph-based assembly listing
4. Analyst effort: Requires assembly knowledge

## Output example

```asm
0x401000 <main>:
    push   rbp
    mov    rbp, rsp
    sub    rsp, 0x20
    mov    DWORD PTR [rbp-0x4], 0
```

## Pros & cons

| Aspect     | Pros                        | Cons                              |
| ---------- | --------------------------- | --------------------------------- |
| Accuracy   | Exact translation of bytes  | Low readability without expertise |
| Detail     | Shows flags, registers, ops | No high-level constructs          |
| Automation | Deterministic, fast         | Limited context inference         |

## Tools

- OSS: radare2, objdump
- Commercial: IDA Pro Disassembler

See tools section. 
