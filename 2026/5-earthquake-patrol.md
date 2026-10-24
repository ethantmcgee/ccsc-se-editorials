# Earthquake Patrol

## Idea

This problem asks for the number of connected components in a graph.  Each "isolated community" is one component, and the answer is how many of them are left standing.

The one twist is that you are handed the graph inside out.  Dispatch reports the streets that are *gone*, not the ones that remain, so what you receive is the complement of the graph you actually care about.  The city starts as a complete graph which is why it begins as a single community.  Two intersections are still directly connected if and only if their street was never reported.

## Approach

Build the complete graph, remove every street that comes in over the wire, and then flood fill what is left.

Start at the first intersection and flood everything reachable from it with a 1.  Then look for the next intersection that has not been filled and flood it with a 2.  Keep going until nothing is left unlabelled.  The last label you handed out is the count you are after, which is why you do not need to tally anything separately.

You do not actually have to build the complete graph.  The only question you ever ask is "is this pair still connected", so keeping the reported pairs in a set and testing `(u, v) not in destroyed` works just as well and skips the construction entirely.  Union-find is a third route, union every pair that was *not* destroyed, then count the distinct roots.

## Complexity

With at most 50 intersections the complete graph holds at most 1225 streets. The flood fill visits each intersection once and looks at every other intersection while it is there, so the whole thing is on the order of n squared, or about 2500 steps.

Recursion is safe here for the same reason: the fill can only nest as deep as there are intersections, so 50 frames at the very worst.  Be aware that this is a property of the small bound and not of the technique.  On a graph with a hundred thousand nodes,  you would want to use union-find instead.

## Solution

```python
intersections = int(input())

graph = {}
for i in range(intersections):
    for j in range(intersections):
        if i not in graph:
            graph[i] = []
        if i != j:
            graph[i] += [j]

try:
    while True:
        u, v = map(int, input().split(" "))
        u, v = u - 1, v - 1
        graph[u].remove(v)
        graph[v].remove(u)
except:
    pass

city = [-1] * intersections

def flood(i, value):
    city[i] = value
    for j in graph[i]:
        if city[j] == -1:
            flood(j, value)

last_flood = 0
for i in graph:
    if city[i] == -1:
        # attempt to flood fill from here
        last_flood += 1
        flood(i, last_flood)

print(last_flood)
```
