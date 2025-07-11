## Reference counting basics
Objective-C uses **retain counts** to manage object lifetime. When an object is created or copied, it starts with a retain count of 1 (owned by the creator). Any part of code that needs to hold onto the object will increment the retain count (retain), and when done will decrement (release). When the retain count drops to zero, the object is deallocated immediately. Under ARC, these retains and releases are inserted by the compiler. For example, in manual terms:

- Creating an object (e.g. `Foo *obj = [[Foo alloc] init]`) gives it retain count 1.
- Assigning it to a strong property or variable may call retain behind the scenes (under MRR) or be handled automatically under ARC.
- When strong references go out of scope or are set to nil, ARC generates the appropriate `release`. Once no strong references remain (count reaches 0), the runtime calls the object’s `dealloc` and [frees its memory](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417).

ARC thus automates what was manual: it “ensures that [objects are deallocated](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417) when they’re no longer needed.” We must still avoid retain cycles (two objects strongly holding each other) which ARC alone can’t fix. That’s where **`__weak` references** (zeroing weak pointers that don’t extend object lifetime) are used to break cycles.
## Autorelease pools
In Objective-C, there is a mechanism for deferred releasing called the **autorelease pool**. An object can be sent an `-autorelease` message, meaning it will be released _later_ instead of immediately. Under the hood, `autorelease` puts the object into the **current autorelease pool** – a container that will send `release` to all its objects at a later time (typically at the end of the event loop iteration). In [Cocoa apps](https://stackoverflow.com/questions/35373183/proper-usage-of-autoreleasepool), the main thread automatically sets up an autorelease pool at the start of each event-loop pass and drains it at the end. As Apple’s documentation [explains](https://stackoverflow.com/questions/35373183/proper-usage-of-autoreleasepool#:~:text=If%20you%20are%20using%20AppKit%2C,have%20to%20create%20autorelease%20pools): “The Application Kit creates an autorelease pool on the main thread at the beginning of every cycle of the event loop, and drains it at the end, thereby releasing any autoreleased objects generated while processing an event.”

This means if you call `[NSString stringWithFormat:@"..."]` inside a button handler, the returned string is autoreleased and will be released when the event handling finishes. Autorelease pools prevent immediate deallocation in situations where an object needs to outlive the method that created it but not much longer. 

We (like from a developer perspective) can also create our own temporary autorelease pools (using `@autoreleasepool { ... }` blocks) around loops or threads that create many autoreleased objects, to free memory sooner than the next UI event tick. Under ARC, the use of autorelease pools is still relevant for managing bursty allocations. (ARC will automatically autorelease objects in some cases, for example, an Objective-C method that returns a new object might insert an autorelease pending the caller retaining it... though ARC has optimizations like `objc_autoreleaseReturnValue` to elide unneeded autoreleases).

## ARC and Core Foundation bridging
Objective-C’s Foundation classes often have **Core Foundation (CF) counterparts** (toll-free bridged types*), and ARC does not directly manage CF objects since they are C types. 

---

\* A [toll-free bridged type](http://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html) is a pair of Core Foundation (C) and Objective-C types that refer to the same underlying object in memory, allowing them to be used interchangeably without conversion or performance overhead.

---

For example, `CFStringRef` and `NSString *` can refer to the same object. The compiler doesn’t automatically know that a `CFTypeRef` should be retained or released; the rules for CF are manual (Create Rule and Get Rule). To handle memory correctly, ARC introduces bridge casts and functions for transferring ownership across the ObjC–CF boundary:

- Using `__bridge` alone will cast without changing ownership (i.e., no retain or release is done). You use this when the memory ownership should not change; for instance, casting an `NSString*` to `CFStringRef` just to call a CF function that doesn’t expect to free it.
- `__bridge_retained` (or the function `CFBridgingRetain`) will **transfer ownership to C** – it takes an ObjC object and returns a CF reference *with a +1 retain count* (ARC will not release it later). You are responsible for releasing that CF object (e.g. via `CFRelease`). This is used when you need to hand an ObjC object to CF code that follows the Create/Copy rule.
- `__bridge_transfer` (or `CFBridgingRelease`) does the opposite: it takes a **CF object and transfers ownership to ARC**. ARC will then treat it as an ObjC object you own, and will release it at the end of the scope. For example, `NSString *name = CFBridgingRelease(CFStringCreate...(...))` lets ARC handle the release of the created CF string.

These bridging annotations are important for proper/good (?) memory management when mixing Core Foundation with Objective-C objects. They prevent leaks and double-frees by clarifying ownership. (If ARC encounters a CF-returning function following the Core Foundation naming conventions--e.g., `Create` or `Copy` in the name--it *might* know it’s owned and require manual release, but in general the above explicit bridges are used to be safe.)

## Runtime behavior and implementation 
The Objective-C runtime implements reference counting efficiently (at least to my knowledge, lol). 

As of 2025, Objective-C (64-bit iOS/tvOS/macOS) uses a **“non-pointer isa”** optimization: instead of each object storing just a class pointer in its `isa` field, the `isa` is a bitfield that can pack additional info such as the object’s [retain count and flags](https://alwaysprocessing.blog/2023/01/19/objc-class-isa) for certain states. In 64-bit, pointers have extra high-order bits available due to alignment and address space limitations. Apple utilizes those bits to store: 

- an **inline reference count** (often called `extra_rc`), and 
- flags like **`has_assoc`** (object has associated objects), 
- **`weakly_referenced`** (there are weak pointers to it), 
- **`has_cxx_dtor`** (it has a C++/ObjC destructor to run), etc.

This means for most objects, the retain count is incremented in a field of the object itself (no separate hash table needed), and only if the count grows beyond what fits in those bits does the runtime resort to an out-of-line storage (a side table). Similarly, if an object gets associated objects (using `objc_setAssociatedObject`) or weak references, the runtime marks those bits and will use auxiliary structures to track them.

When an object is deallocated (either because ARC determined it’s unreachable or manual `release` brought count to zero), the runtime checks those flags to decide the deallocation path. There is a **fast deallocation path**: if the object has no associated objects, no weak refs, and no special cleanup needed, it can skip some housekeeping. In fact, in Apple’s open-source runtime you can see:

```c
// code from "alwaysprocessing.blog"
if (isa().nonpointer &&            // using non-pointer isa
    !isa().weakly_referenced && 
    !isa().has_assoc && 
    !isa().has_cxx_dtor && 
    !isa().has_sidetable_rc) {
        free(this);               // directly free object memory
} else {
        object_dispose((id)this); // do full cleanup, then free
}
```

If none of the extra work is needed, the object’s memory is immediately freed with `free()`. Otherwise, `object_dispose` will do things like zero out any weak reference entries, remove associated objects, call C++ destructors (`.cxx_destruct` methods) to clean up C++ ivars, etc., then free the memory. 

This design optimizes the common case (objects without weak refs or associations) while still correctly handling the more complex cases. Notably, if an object’s **retain count** overflowed the inline capacity (rare, only if an object was legitimately retained thousands of times or maliciously so), the `has_sidetable_rc` bit is set and the extra counts live in a side table structure; the runtime will consult and clear that on `object_dispose`.

## Autorelease implementation
Autorelease pools are implemented as simple stacks of pointers. An `@autoreleasepool` block in ARC is translated into calls to `objc_autoreleasePoolPush()` at the start and `objc_autoreleasePoolPop()` at the end. When an object is sent `autorelease`, it’s added to the latest pool. 

Draining a pool (pop) calls `release` on each object in that pool. Under ARC, the compiler often optimizes away autoreleases for return values (using LLVM’s “return value optimization” known as ARV* and RRVs*) to avoid unnecessary object churn. Still, understanding that autoreleased objects will live until the pool is drained is important for performance... hence patterns like [wrapping tight loops](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417) in their own autorelease pool to prevent a large transient memory spike.

---

\* ARV (autorelease return value) and RRV (retain return value) are compiler/runtime optimizations in Objective-C that manage how returned objects are retained or autoreleased to eliminate unnecessary memory operations and improve performance.

---
