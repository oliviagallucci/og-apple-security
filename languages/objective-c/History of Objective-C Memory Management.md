Notes: https://en.wikipedia.org/wiki/Objective-C 
## 1980s origins and early model
Objective-C was created in the early 1980s (by Brad Cox and Tom Love) as an object-oriented layer on C, adopting Smalltalk-style messaging. 

Early Objective-C used **manual reference counting** – developers called `retain` (to take ownership) and `release` (to relinquish ownership) on objects. This **Manual Retain-Release (MRR)** model was the norm through the 1990s and 2000s. It provided deterministic memory management but relied on programmer discipline; forgetting a `release` caused [leaks](https://medium.com/@melissazm/advanced-memory-management-in-ios-exploring-arc-manual-retain-release-and-memory-leaks-f5c69ed68417), while an extra `release` caused crashes.

