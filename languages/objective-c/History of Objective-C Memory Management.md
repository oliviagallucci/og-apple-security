Notes: https://en.wikipedia.org/wiki/Objective-C 
## 1980s origins and early model
Objective-C was created in the early 1980s (by Brad Cox and Tom Love) as an object-oriented layer on C, adopting Smalltalk-style messaging. 

Early Objective-C used **manual reference counting** – developers called `retain` (to take ownership) and `release` (to relinquish ownership) on objects. This **Manual Retain-Release (MRR)** model was the norm through the 1990s and 2000s. It provided deterministic memory management but relied on programmer discipline; forgetting a `release` caused [leaks](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417), while an extra `release` caused crashes.

## 2006 Obj-C 2.0 and garbage collection
In 2006, Apple announced _Objective-C 2.0_, introducing modern features and an optional **garbage collector** on Mac OS X (desktop). Under GC, `retain/release` calls became no-ops and a background thread automatically reclaimed objects (using a conservative generational collector). A zero-ing weak reference system was added (so `__weak` pointers would nil out on object deallocation). Notably, this **GC was never available on iOS** – iOS always stuck to explicit reference counting. Garbage collection on macOS proved to have performance and complexity trade-offs, and Apple **deprecated GC in OS X 10.8 (2012)** in favor of a new compile-time approach called ARC.
