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

\* A toll-free bridged type is a pair of Core Foundation (C) and Objective-C types that refer to the same underlying object in memory, allowing them to be used interchangeably without conversion or performance overhead.

---

For example, `CFStringRef` and `NSString *` can refer to the same object. The compiler doesn’t automatically know that a `CFTypeRef` should be retained or released; the rules for CF are manual (Create Rule and Get Rule). To handle memory correctly, ARC introduces bridge casts and functions for transferring ownership across the ObjC–CF boundary:

- Using `__bridge` alone will cast without changing ownership (i.e., no retain or release is done). You use this when the memory ownership should not change; for instance, casting an `NSString*` to `CFStringRef` just to call a CF function that doesn’t expect to free it.
- `__bridge_retained` (or the function `CFBridgingRetain`) will **transfer ownership to C** – it takes an ObjC object and returns a CF reference *with a +1 retain count* (ARC will not release it later). You are responsible for releasing that CF object (e.g. via `CFRelease`). This is used when you need to hand an ObjC object to CF code that follows the Create/Copy rule.
- `__bridge_transfer` (or `CFBridgingRelease`) does the opposite: it takes a **CF object and transfers ownership to ARC**. ARC will then treat it as an ObjC object you own, and will release it at the end of the scope. For example, `NSString *name = CFBridgingRelease(CFStringCreate...(...))` lets ARC handle the release of the created CF string.

These bridging annotations are important for proper/good (?) memory management when mixing Core Foundation with Objective-C objects. They prevent leaks and double-frees by clarifying ownership. (If ARC encounters a CF-returning function following the Core Foundation naming conventions--e.g., `Create` or `Copy` in the name--it *might* know it’s owned and require manual release, but in general the above explicit bridges are used to be safe.)

s