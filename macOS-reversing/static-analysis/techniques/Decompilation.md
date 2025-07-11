# Decompilation

Decompilation lifts machine code into high-level source approximations (e.g., C/C++), enabling quicker comprehension of program logic.

Inferring high-level constructs (types, loops, functions) from compiled binaries using control- and data-flow analysis.

Uses: 

* fast program auditing and code review
* recover lost source for large binaries
* Identify high-level algs and control flow

## Workflow (?)

1. Input: Binary executable or library
2. Engine: CFG reconstruction + data-flow heuristics
3. Output: Pseudo-source code with variables and structures
4. Analyst effort: Moderate; often requires manual cleanup

## Output example

```c
int main(void) {
    int counter = 0;
    while (counter < SOME_LIMIT) {
        // ...
        counter++;
    }
    return 0;
}
```

## Pros & cons

| Aspect        | Pros                                    | Cons                             |
| ------------- | --------------------------------------- | -------------------------------- |
| Readability   | Resembles original source, easy to scan | May introduce inaccuracies       |
| Automation    | Quick overview of logic                 | Heuristics can misidentify types |
| Type Recovery | Infers variables and data structures    | Abstracts low-level nuances      |

## Tools

- OSS: Ghidra, RetDec
- commercial: Hex-Rays Decompiler

See tools section. 
