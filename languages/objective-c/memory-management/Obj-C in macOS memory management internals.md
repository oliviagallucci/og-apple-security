Objective-C is deeply ingrained in macOS and iOS; it doesn’t operate in isolation but on top of OS memory management and security mechanisms. 

That said, I think there are some interesting aspects of how the Obj-C runtime interacts with low-level systems like allocators, and how it’s influenced by (and contributes to) OS security. 
## Allocation and deallocation (malloc integration)
Objective-C object allocation ultimately uses the standard C memory allocator. When you call `[[MyClass alloc] init]`, the runtime will ask the class for its instance size, add any alignment or metadata padding, and allocate that many bytes. In the era of NSZone (historic allocator zones), you could specify a zone, but now **zones are ignored**... all allocations go to the default malloc zone.

`+allocWithZone:` is implemented to just call `+alloc` and is kept only for API compatibility. The memory comes from heap segments managed by `malloc`. Apple’s malloc on macOS/iOS uses various heuristics (tiny, small, large buckets, etc.), and Objective-C doesn’t need any special support from it beyond regular allocation and free.

The Obj-C runtime does have its own small allocator for certain metadata (for example, runtime-created classes or blocks may use `calloc` directly, and associated object keys use a side table that is essentially a hash map). But _object_ instances are malloc’ed. Deallocation of an object ends up in a call to `free()` (either directly or via `object_dispose` after cleanup). There is also an environment variable `MallocStackLogging`; when enabled, can help debug memory issues including Objective-C objects (and tools like Zombies: setting `NSZombiesEnabled` causes freed Obj-C objects to turn into zombie stubs* rather than actually freeing, to help detect overreleases).

---

\* Zombie stubs are placeholder objects used in debugging that replace deallocated Objective-C objects to catch use-after-free errors by crashing the program when a message is sent to the zombie.

---

In the past, Apple’s GC used a custom allocator (“Auto Zone”) that coexisted with malloc – but now that’s gone. Thus, from the OS perspective, an Objective-C program’s memory usage is just normal heap allocations (plus some small contributions from the Objective-C runtime for metadata).
