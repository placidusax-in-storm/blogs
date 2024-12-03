**Find Bottom Left Tree Value**

Invariant: same level nodes are push from right to left, so does deque. Like DFS.

Using `queue.length` to check whether is queue empty.

`unshift` and `visted = node.val` perceive as visiting.

**Longest Palindromic Substring**

not DP is ok. We observe that a palindrome mirrors around its center. Therefore, a palindrome can be expanded from its center, and there are only 2n - 1 such centers.

Quick sort

Pick rightest as pivot, move all less to left, all greater to right which call partition then

quick sort left part and right part.

For partition, there is a key index “stored”, which start from leftest - 1, maintain the index of pushed numbers for which is less than pivot.

In the loop of partition which from leftest to rightest - 1, if number less than pivot, push it into stored by `stored` + 1 and swap that into `stored` + 1.

After loop, swap the `stored` + 1 and the pivot to “cover” it.

Jump game 1

Iterate from last, truncate out which is reachable to the last reached.

divisorGame

```jsx
/**
 * @param {number} n
 * @return {boolean}
 */
var divisorGame = function(n) {
    // n
    // i in (n, 1]: if n % i === 0
    //  choice space is [in, i0]
    //    question: which is optimally
    // j in [in, i0] return f(x - j)
    // caching?
};
```