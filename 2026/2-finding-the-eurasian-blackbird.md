# Finding the Eurasian Blackbird

## Idea

At a glance this looks like the famous shortest path problem, but it asks the opposite question, what is the *longest* path, and that difference matters far more than it first appears.  Shortest path has efficient algorithms like Dijkstra.  Longest *simple* path does not: it is NP-hard.

Greedy choices are not safe either.  Consider this graph, going from 0 to 4:

```
0 1 5.0
1 4 1.0
0 2 1.0
2 3 5.0
3 4 5.0
```

Taking the longest edge first (0 to 1, length 5.0) reaches the end with a total of only 6.0.  The actual longest path begins with the *shortest* edge out of 0 and reaches 11.0 by way of 0, 2, 3, 4.  You cannot decide the first step without exploring where each choice leads.

## Approach

Enumerate every simple path from the start to the end and keep the best one.  Carry the set of vertices already on the current path along with it; when you consider a neighbour that is already on that path, skip that one neighbour and keep exploring the others.

There are two things to consider as you search though:

**Probability.**  You need the chance of seeing a Blackbird *at least once*, which is not the sum of the edge probabilities.  The cleanest way is to track the probability of seeing nothing at all and subtract at the end:

```
P(at least one) = 1 - (1 - p1)(1 - p2)...(1 - pn)
```

or to accumulate it one edge at a time:

```
combined = current + edge - current * edge
```

**Tie-break.**  When several paths tie for longest, you must output the one with the higher probability.  Sorting the finished paths by `(length, probability)` and taking the largest satisfies both rules at once.

## Complexity

Exponential in the worst case, which is fine at no more than 50 edges.  Directed edges help a great deal here, because the branching factor is far lower than it would be undirected.

But directed does *not* mean acyclic.  Several graphs in this problem contain directed cycles, which is why the visited set is not optional.

## Solution

```python
edges, start, end = map(int, input().split(" "))

graph = {}
for _ in range(edges):
    s, e, d, p = map(float, input().split(" "))
    s, e = int(s), int(e)

    if s not in graph:
        graph[s] = {}
    graph[s][e] = (d, p)

to_continue = [(start, 0, 0, [start])]
finished_cases = []

while len(to_continue) > 0:
    new_to_continue = []
    for case in to_continue:
        n1, cur_d, cur_p, path = case
        for n2, values in graph.get(n1, {}).items():
            edge_d, edge_p = values
            if n2 == end:
                # we reached the end
                finished_cases += [(cur_d + edge_d, cur_p + edge_p - cur_p * edge_p, path + [n2])]
            elif n2 not in path:
                # as long as we've not been here before
                new_to_continue += [(n2, cur_d + edge_d, cur_p + edge_p - cur_p * edge_p, path + [n2])]
    to_continue = new_to_continue

finished_cases = sorted(finished_cases)[::-1]

print("Length: %.2f" % finished_cases[0][0])
print("Probability: %.2f%%" % (finished_cases[0][1] * 100))
```
