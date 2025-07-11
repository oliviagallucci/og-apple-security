## Reference counting basics
Objective-C uses **retain counts** to manage object lifetime. When an object is created or copied, it starts with a retain count of 1 (owned by the creator). Any part of code that needs to hold onto the object will increment the retain count (retain), and when done will decrement (release). When the retain count drops to zero, the object is deallocated immediately. Under ARC, these retains and releases are inserted by the compiler. For example, in manual terms:

- Creating an object (e.g. `Foo *obj = [[Foo alloc] init]`) gives it retain count 1.
- Assigning it to a strong property or variable may call retain behind the scenes (under MRR) or be handled automatically under ARC.
- When strong references go out of scope or are set to nil, ARC generates the appropriate `release`. Once no strong references remain (count reaches 0), the runtime calls the object’s `dealloc` and [frees its memory](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417).

ARC thus automates what was manual: it “ensures that [objects are deallocated](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417) when they’re no longer needed.” We must still avoid retain cycles (two objects strongly holding each other) which ARC alone can’t fix. That’s where **`__weak` references** (zeroing weak pointers that don’t extend object lifetime) are used to break cycles.

