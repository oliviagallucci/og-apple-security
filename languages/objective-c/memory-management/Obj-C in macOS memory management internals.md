# Obj-C in macOS memory management internals
Objective-C is deeply ingrained in macOS and iOS; it doesn’t operate in isolation but on top of OS memory management and security mechanisms. 

That said, I think there are some interesting aspects of how the Obj-C runtime interacts with low-level systems like allocators, and how it’s influenced by (and contributes to) OS security. 
## Allocation and deallocation (malloc integration)
Objective-C object allocation ultimately uses the standard C memory allocator. When you call `[[MyClass alloc] init]`, the runtime will ask the class for its instance size, add any alignment or metadata padding, and allocate that many bytes. In the era of NSZone (historic allocator zones), you could specify a zone, but now **zones are ignored**... all allocations go to the [default malloc zone](https://langdev.stackexchange.com/questions/2370/why-did-objective-c-remove-nszone).

`+allocWithZone:` is implemented to just call `+alloc` and is kept only for API compatibility. The memory comes from heap segments managed by `malloc`. Apple’s malloc on macOS/iOS uses various heuristics (tiny, small, large buckets, etc.), and Objective-C doesn’t need any special support from it beyond regular allocation and free.

The Obj-C runtime does have its own small allocator for certain metadata (for example, runtime-created classes or blocks may use `calloc` directly, and associated object keys use a side table that is essentially a hash map). But _object_ instances are malloc’ed. Deallocation of an object [ends up in a call](https://alwaysprocessing.blog/2023/01/19/objc-class-isa) to `free()` (either directly or via `object_dispose` after cleanup). There is also an environment variable `MallocStackLogging`; when enabled, can help debug memory issues including Objective-C objects (and tools like Zombies: setting `NSZombiesEnabled` causes freed Obj-C objects to turn into zombie stubs* rather than actually freeing, to help detect overreleases).

---

\* Zombie stubs are placeholder objects used in debugging that replace deallocated Objective-C objects to catch use-after-free errors by crashing the program when a message is sent to the zombie.

---

In the past, Apple’s GC used a custom allocator (“Auto Zone”) that coexisted with malloc – but now that’s gone. Thus, from the OS perspective, an Objective-C program’s memory usage is just normal heap allocations (plus some small contributions from the Objective-C runtime for metadata).

## Memory zones (NSZone)
As noted, NSZone was a mechanism for custom memory allocation pools. You could create an NSZone to allocate a bunch of objects together, perhaps to improve locality or facilitate bulk deallocation. This was a Mach-era concept that never proved very useful and is essentially deprecated. 

At present (afaik), Objective-C runtime just ignores the zone parameter and uses the default heap. Apple explicitly [states](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW14): “There is no need to use NSZone any more. They are ignored by the modern Objective-C runtime.” So, Objective-C doesn’t really integrate with malloc “zones” in any special way today. Tools like **Guard Malloc** (for debugging buffer overruns) or **malloc nano/secure modes** apply to Objective-C objects the same as any heap allocation.
## Tagged pointers
One interesting memory optimization in Objective-C (since iOS 7 / OS X Mavericks era) is **tagged pointers**. Certain classes--notably NSNumber, NSDate, and a few others (even NSString for small strings on recent systems)--don’t use malloc at all for certain values. Instead, the pointer value itself is encoded with the data. 

For example, instead of allocating a full NSNumber object for an integer 42, the runtime can represent it as a pointer where some bits actually store the integer value and a tag that tells the runtime “this isn’t a real heap pointer, it’s an encoded number.” These tagged pointers have a special bit pattern (usually LSB=1 or a specific high bit set) so that the runtime recognizes them. When an API like `[NSNumber numberWithInt:42]` is called, you might get a tagged pointer (no heap allocation, extremely fast, no need to free). The `objc_msgSend` knows to check for tagged pointers and, if found, treat them differently (in fact, many operations on them are implemented by simply checking the pointer bits). This reduces memory usage and fragmentation for lots of small immutable objects. 

Tagged pointers are an interplay* between the runtime and CPU addressing (it uses the fact that addresses are aligned and some bits are unused). 

---

\* not a security term, but an interplay is the dynamic interaction or mutual influence between two or more elements that affect each other’s behavior or outcome.

---

From a security perspective, tagged pointers are **non-pointer data masquerading as pointers**, which could confuse traditional exploit techniques. But nowadays, pointer authentication also helps here (PAC has to ignore those or treat the tag bits appropriately). 

If an exploit tried to forge a tagged pointer, they’d have to ensure the bits matched a valid encoding or the runtime would just treat it as a huge odd pointer and likely crash on access. In short ig, **tagged pointers improve performance and memory usage** for certain Obj-C objects by avoiding malloc entirely... an example of runtime and memory allocator integration for optimization.


