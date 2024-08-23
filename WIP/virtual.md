How the lib handle dynamic row height?

how do you support search?

how do you optimize the performance?

Structure:

Pass `scrollElementRef` to `useVirtualizer` and then get visible items using `getVirtualItems`.

Visible items are absolutely positioned according to `scrollElement` using transformY, which is obtained from `virtualItem`.

In the case of dynamic row height, the `measureElement` should be passed to every virtualElement.

How `getVirtualItems` determine which item are visible?

1. Fixed row height: by offset? Trivial detail, no need to dive.
2. Dynamic row height

How to get height of Monaco diff-editor?