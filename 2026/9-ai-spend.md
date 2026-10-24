# AI Spend

## Idea

Every server goes north or south.  Each choice earns a profit, and each pair that ends up split costs a premium.  The `3` per server is the same no matter what you do, so it is a constant to subtract at the very end and plays no part in the decision.

The instinct is to be greedy: put each server where it earns more, then see what the premiums come to.  The sample is built to punish that.  Greedy puts server 1 north and server 2 south, which is 20 units of profit, and then pays a 20-unit premium for splitting them, on top of a 1-unit premium for splitting 3 and 4.  Net result: **−3**.  The optimal play drags server 2 north at a cost of 9 in lost profit, but buys its way out of the 20-unit premium, and nets **8**.

Trying every assignment is `2^n`, and `n` goes to 200.  So you need something that reasons about all the choices together.

This is a **minimum cut** problem.  Threre are several algorithms for handling like Ford Fulkerson, Edmunds-Karm, and Dinic.  All will work on this particular problem.

## Approach

### Turn maximising into minimising

A cut is something you minimise.  The problem asks you to maximise.  So flip it.  Instead of counting the profit you earn, count the profit you **forgo**: for a server placed north, the south profit you gave up, and vice versa.

```
profit  =  Σ (chosen-side profit)  −  3n  −  Σ premiums paid
loss    =  Σ (other-side profit)         +  Σ premiums paid
```

For every server, chosen-side plus other-side is just `north + south`.  So `profit + loss` is a constant, and the assignment that minimises loss is the one that maximises profit:

```
answer  =  Σ (north_i + south_i)  −  3n  −  min loss
```

### Build the network

Source `S` means "north", sink `T` means "south".  After the cut, a server on the source side is north and one on the sink side is south.

- **`S → i` with capacity `north_i`.**  This edge is cut when `i` lands on the sink side — that is, when `i` goes south — and paying `north_i` is exactly the profit forgone.
- **`i → T` with capacity `south_i`.**  Cut when `i` goes north; pays the forgone `south_i`.
- **`x ↔ y` with capacity `z`, in both directions.**  Cut exactly when `x` and `y` are on opposite sides, and pays `z` once.  Both directions are needed because you do not know in advance which one will be on the source side.

Each server is a single node.  Two extra nodes for `S` and `T`.  Two edges per server plus two per pair: at most 202 nodes and roughly 20,400 edges.

### Compute the min cut

By the max-flow / min-cut theorem, the minimum cut equals the maximum flow from `S` to `T`.  So run any max-flow algorithm:

- **Ford–Fulkerson**: repeatedly find any augmenting path and push flow along it.
- **Edmonds–Karp**: the same, but always take the *shortest* augmenting path (found by BFS), which bounds the number of iterations.
- **Dinic**: builds a level graph and pushes a blocking flow through it each phase.  Fastest, and the standard choice in a library.

The graph here is small enough that all three are comfortably fast.  The official solution is Edmonds–Karp and needs at most 456 augmenting paths on the largest judge case, about a tenth of a second in Python.

Then:

```python
print(sum(north) + sum(south) - 3 * n - max_flow)
```

### Why it is allowed to work

The premium is a penalty for **disagreeing** and never a bonus.  That is the condition (called *submodularity*) that lets a pairwise term be expressed as a cut.  If the problem allowed a negative `z` — a reward for splitting a pair — you would need a negative-capacity edge, and this whole approach falls apart.  The statement's `z ≥ 0` is precisely the guarantee you are leaning on.

## Complexity

Building the graph is `O(n + k)`.  Edmonds–Karp is `O(V · E²)` in the worst case, which sounds alarming at `E ≈ 20,000`, but the bound is loose and the actual number of augmenting paths on this data is a few hundred.  Dinic on a graph this size is effectively instant.

## Solution

```python
from collections import deque

num_servers, penalty_ct = map(int, input().split(" "))
north_profit = list(map(int, input().split(" ")))
south_profit = list(map(int, input().split(" ")))

def bfs(graph, s, t):
    parent = {s: None}
    q = deque([s])
    while q:
        u = q.popleft()
        for v, cap in graph[u].items():
            if cap > 0 and v not in parent:
                parent[v] = u
                if v == t:
                    path = [t]
                    while parent[path[-1]] is not None:
                        path.append(parent[path[-1]])
                    return path[::-1]
                q.append(v)
    return None

penalties = []
for _ in range(penalty_ct):
    x, y, cost = map(int, input().split(" "))
    # normalize to 0 based
    penalties += [(x - 1, y - 1, cost)]

# build the residual grab
graph = {i: {} for i in range(num_servers)}
graph[-1] = {} # start node
graph[-2] = {} # end node

for i in range(num_servers):
    graph[-1][i] = north_profit[i]
for i in range(num_servers):
    graph[i][-2] = south_profit[i]
for penalty in penalties:
    x, y, cost = penalty
    graph[x][y] = cost
    graph[y][x] = cost

# now run edmunds-karp
possible_path = bfs(graph, -1, -2)
flow = 0
while possible_path != None:
    # analyze path, get min_cost
    last_min_cost = 999999999
    min_pos = -1
    last = -1
    for i in range(1, len(possible_path)):
        new_min_cost = min(last_min_cost, graph[last][possible_path[i]])
        if new_min_cost != last_min_cost:
            min_pos = i
            last_min_cost = new_min_cost
        last = possible_path[i]

    last = -1
    for i in range(1, len(possible_path)):
        # for everything prior to the min_pos, subtract the min, add back path
        # for everything after min_pos, subtract the min, add back path
        if i != min_pos:
            orig = graph[last][possible_path[i]]
            graph[last][possible_path[i]] -= last_min_cost
            if last in graph[possible_path[i]]:
                graph[possible_path[i]][last] += last_min_cost
            else:
                graph[possible_path[i]][last] = last_min_cost
        # for min_pos, flip
        else:
            del graph[last][possible_path[i]]
            if last in graph[possible_path[i]]:
                graph[possible_path[i]][last] += last_min_cost
            else:
                graph[possible_path[i]][last] = last_min_cost
        last = possible_path[i]

    flow += last_min_cost
    possible_path = bfs(graph, -1, -2)

print(sum(north_profit) + sum(south_profit) - 3 * num_servers - flow)
```
