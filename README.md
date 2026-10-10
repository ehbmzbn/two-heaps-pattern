# Two Heaps Pattern: Recognize Median Problems on Sight, Keep Both Heaps Balanced, and Write the Template Under Pressure

A problem asks you to return the median after every insertion into a stream. You write something that sorts the whole thing each time, watch the time limit blow up, and then read the editorial: two heaps. That moment is annoying, because the fix is only about eight lines of code.

The two heaps pattern is small enough to memorize in an afternoon. The hard part is not the code — it is recognizing in the first thirty seconds that a problem wants two heaps instead of a single heap, a sorted list, or a balanced BST. That recognition step is what interviewers are actually scoring, and it is what this article is mostly about.

## What the two heaps pattern really is

Two priority queues pointed at the same data set:

- A **max-heap** holding the smaller half of everything seen so far. Its root is the largest value in that half — the element closest to the middle from the left.
- A **min-heap** holding the larger half. Its root is the smallest value in that half — the closest element from the right.

Everything else is bookkeeping. Two rules keep the structure honest:

1. Every element in the max-heap is less than or equal to every element in the min-heap. In practice you only compare roots, because the root is what each heap guarantees.
2. The two sizes differ by at most one element.

With those rules in place, reading the median is free:

- Sizes equal → the median is the average of the two roots.
- Max-heap one larger → the median is the max-heap root.

Why bother instead of keeping a sorted array? Because you never need the sorted order of the whole data set. You need the one or two values sitting at the boundary between halves, and a heap gives you exactly that at `O(log n)` per insertion instead of `O(n)` for shifting elements around in a list.

Costs, once built:

| Operation | Time | Why |
| --- | --- | --- |
| Insert a number | O(log n) | One heap push plus at most one transfer |
| Find the median | O(1) | Heap roots, no traversal |
| Rebalance | O(log n) | At most one element moves between heaps |
| Memory | O(n) | Every element lives in exactly one heap |

That complexity table is worth being able to write on a whiteboard from memory, because the follow-up question after "how does it work" is almost always "what does insertion cost, and why".

## The signals that mean two heaps — and the ones that don't

Most two-heaps problems announce themselves with a small set of phrases. The pattern only fits when you need to repeatedly query a partition boundary of a changing data set.

| What the problem says | What it usually wants |
| --- | --- |
| "median after each insertion", "running median" | Two heaps, no removal |
| "median of every window of size k" | Two heaps plus lazy deletion |
| "balance two groups", "partition into lower and upper half" | Two heaps |
| "maximize capital by picking projects you can afford" | Min-heap of requirements + max-heap of gains (two structures, greedy pairing) |
| "minimum number of rooms / machines needed" | Min-heap of end times, sometimes two heaps |
| "kth largest" or "top k" in a static array | One heap of size k — not this pattern |

The last row matters. A single heap of size k solves the kth-largest class of problems. Two heaps solve the boundary-of-a-changing-set class. They are related because both exploit the same property — only the root is sorted — but they are not the same template, and mixing them up costs you the explanation even if the code passes.

There is also a negative space worth practicing deliberately. "Find the maximum value in this array" is not a heap problem. Dijkstra's algorithm uses a heap, but the answer an interviewer wants is "shortest path with a priority queue", not "heap pattern". Recognizing when a heap appears without being the point is the difference between naming the right tool and naming a tool.

## The three-step template

Every version of two heaps follows the same three steps.

**1. Route the new value.** If the max-heap is empty, or the value is smaller than the max-heap root, push it to the max-heap. Otherwise push it to the min-heap.

**2. Rebalance.** After the push, check sizes. If the max-heap has grown more than one element larger than the min-heap, move its root over. If the min-heap has become strictly larger, move its root back.

**3. Read.** Equal sizes → average of the two roots. Max-heap larger → max-heap root.

Python's `heapq` is a min-heap only, so the lower half stores negated values. A common shortcut, and the one used in the editorial for LeetCode 295, routes through `heappushpop`:

python
import heapq

class MedianFinder:
    def __init__(self):
        self.lower = []   # max-heap via negation
        self.upper = []   # min-heap

    def addNum(self, num: int) -> None:
        heapq.heappush(self.upper, -heapq.heappushpop(self.lower, -num))
        if len(self.upper) - len(self.lower) > 1:
            heapq.heappush(self.lower, -heapq.heappop(self.upper))

    def findMedian(self) -> float:
        if len(self.lower) == len(self.upper):
            return (self.upper[0] - self.lower[0]) / 2.0
        return float(self.upper[0])


The trick in `addNum` is that the value goes into the max-heap first, and whatever comes back out is by construction the largest of the lower half — then it is pushed onto the upper heap. One push, one pop, ordering maintained, rebalancing limited to a single extra move.

Language notes that come up in real interviews:

- **Java**: two `PriorityQueue`s, the lower half built with `Collections.reverseOrder()`. No negation needed, but be ready to explain the comparator.
- **C++**: `priority_queue<int, vector<int>, greater<int>>` for the min-heap; the default `priority_queue` is already a max-heap.
- **JavaScript**: no built-in heap in the standard library, which is why so many JS solutions to LeetCode 295 hand-roll a `Heap` class with comparators and bubble-up/bubble-down methods.

If the interviewer allows using a library heap, take it. If they want the heap written from scratch, the answer is still the same two heaps — you just spend five minutes on the plumbing.

## Where two heaps gets harder

The clean template above handles a growing data set. Three variants are the ones that actually separate candidates.

### Sliding window median (LeetCode 480)

Now elements also leave the window. Removing an arbitrary element from a heap costs a linear scan to find it, which turns a clean `O(log n)` per step into something much worse. The standard fix is lazy deletion: keep a hash map of values that should be gone, and only clean the top of a heap when the stale value has reached the root and would otherwise distort the median read. The two heaps and the balance invariant do not change; what changes is that staleness has to be filtered out at read time.

### Two heaps plus greedy (LeetCode 502, maximizing capital)

This is a different shape. Sort projects by the capital they require, push all projects you can currently afford into a max-heap keyed by profit, pop the best one, add its profit to your capital, and repeat. The min-heap holds requirements, the max-heap holds gains, and a greedy choice links them. If you have only seen the median version of the pattern, this problem looks unrelated; if you have internalized "two ordered views of the same data", it looks like the same idea wearing different clothes.

### Intervals and scheduling

Meeting Rooms II keeps a min-heap of end times so you can ask "does anything free up before this meeting starts". Meeting Rooms III goes further and is typically solved with two min-heaps — one for available rooms ordered by number, one for rooms currently in use ordered by the time they free up — plus a counter array. The task-scheduling-on-minimum-machines family works the same way. Task Scheduler (LeetCode 621) is usually described as a max-heap of remaining frequencies plus a cooldown queue; the heap is doing the heavy lifting, the queue is not a second heap, and saying so out loud is better than forcing it into the two-heaps label.

## Alternatives you should be ready to compare against

Being able to explain why two heaps beats the alternatives is worth more than a perfect implementation.

- **Sorted list with binary insert.** Insertion is `O(n)` because of shifting, and the median is `O(1)` to read. Fine for small inputs, dead on a stream of a million numbers.
- **Balanced BST or an order-statistics tree.** Insertion `O(log n)`, median lookup `O(log n)`. Correct, and often the "real" answer in production systems, but most interview languages make you hand-roll it, which is a bigger time sink than two heaps.
- **Sort once, then query.** Works only if the data set is static. A problem that says "stream" is telling you it is not.
- **Quickselect.** Good for a single kth-element query on a fixed array, useless when new numbers keep arriving.

The honest interview answer usually sounds like: two heaps give `O(log n)` insertion and `O(1)` median reads, at the cost of `O(n)` memory; a balanced BST gives `O(log n)` for both and supports arbitrary order statistics, which matters if the follow-up asks for the 90th percentile rather than the median. Naming that trade-off is what makes the pattern knowledge look like understanding rather than recall.

## A practice order that builds the recognition

Working the same pattern in increasing difficulty beats grinding random heap problems. A sequence that covers the whole family:

1. **LeetCode 295, Find Median from Data Stream.** Learn the invariant and the rebalancing logic. Do it in two languages, because the negation trick and the comparator approach are both fair game.
2. **LeetCode 973, K Closest Points to Origin.** A single heap of size k, included here so you can feel the difference between the two templates.
3. **LeetCode 253, Meeting Rooms II.** Heap over end times. Resources being released and re-acquired.
4. **LeetCode 480, Sliding Window Median.** The lazy-deletion variant. Expect to get this wrong the first time; the bug is almost always a stale root leaking into the median read.
5. **LeetCode 502, IPO / Maximize Capital.** Two structures plus a greedy choice.
6. **LeetCode 621, Task Scheduler.** Where the heap is one component of the solution rather than the whole solution.

Six problems, one pattern family, and most of the interview surface around heaps. After that, adding volume is about speed, not concepts.

If you want the pattern explained with the decision logic laid out before the code, AlgoMonster covers heaps as its own pattern category alongside the other 47, with written walkthroughs and diagrams for each problem rather than videos — 👉 [see how the heap pattern is structured on AlgoMonster](https://algo.monster?fpr=li69). Their own interview-stats page rates priority queue/heap as medium ROI and names exactly the classic cases: the median of a data stream and k closest points. There is also a decision flowchart that routes a "kth smallest or largest?" question straight into the heap branch, which is a useful cross-check on your own recognition instinct.

## What AlgoMonster costs, and who it actually fits

AlgoMonster is a pattern-first course built by ex-Google engineers. Instead of a several-thousand-problem bank, it teaches a fixed set of patterns with a code template and a handful of variants each. The current site copy advertises all patterns, lessons and problems unlocked, unlimited AI assistant usage, unlimited Speedrun (timed practice), real interview questions from top companies refreshed monthly, plus bonus courses covering CS fundamentals and building an LLM from scratch.

Pricing has moved around a lot with promotions, so here is what the plans look like as currently listed:

| Plan | What you get | Price as last checked | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | A limited subset of patterns and problems — enough to judge the teaching style, not enough to prep with | $0 | — | [Start with the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | Everything unlocked: all patterns, lessons and problems, unlimited AI assistant, unlimited Speedrun, company question sets updated monthly, bonus courses, member discount on live bootcamps | $69.99/year (listed as $5.84/month billed annually) | 12 months, then renews | [Check the Pro Yearly price](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan plus lifetime updates | $89 one-time | One payment | [Check the Pro Lifetime price](https://algo.monster?fpr=li69) |

Two things to know before you click. There is no monthly plan — it is annual or lifetime, nothing in between. And AlgoMonster runs frequent sales, which is why third-party writeups have quoted the yearly plan anywhere from about $99 to $119 and lifetime access from $189 to $459 at various points. The number that matters is the one at checkout; both paid plans sit on the same subscribe page, so you are choosing between two options, not hunting for a separate product page.

What it does not do is worth saying plainly. The platform is text and diagrams only, with no video walkthroughs at all, which is fine if you learn by reading and immediately coding but genuinely worse if you need someone talking through a solution. The problem count is in the low hundreds, so if your gap is raw volume and speed on unseen problems, you will still want a larger bank alongside it. System design material is introductory, which is not enough for senior-level loops. And every problem is solved at your own pace with no timer and no interviewer probing your reasoning, so timed practice has to come from somewhere else — Speedrun helps, but it is not a mock interview.

Where it fits well is narrower and clearer: you can already write working code in one language, your bottleneck is selecting the right approach rather than implementing it, and you have four to eight weeks with an hour or two a day. That is precisely the situation where a pattern like two heaps stops being a thing you recognize in the editorial and starts being a thing you recognize in the first minute.

## Quick answers people search for

**Is two heaps the same as a priority queue?** A heap is the data structure; a priority queue is the interface. Two heaps means two of them, used to maintain the boundary between a lower and an upper half.

**Can two heaps handle removal?** Not cleanly. Heap deletion needs a linear scan to locate the element, so real solutions use lazy deletion with a counter or hash map and clean the roots only when stale values surface.

**Do I need two heaps for kth largest?** No. A single min-heap of size k does it in `O(n log k)`. Two heaps are for problems where the data set keeps changing and you need the middle.

**What is the space cost?** `O(n)` — every element lives in one of the two heaps, and nothing is discarded in the plain streaming version.

The pattern itself is short. Learn the invariant, keep the sizes within one, and let the follow-up questions about complexity and trade-offs do the rest of the work.

👉 [Compare the Pro Yearly and Pro Lifetime plans before the next sale ends](https://algo.monster?fpr=li69)
