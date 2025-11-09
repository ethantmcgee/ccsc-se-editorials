# Constellation Counter

This problem is a classic application of finding connected components in a graph. The stars are the nodes (vertices), and the connections between them are the edges. A "star cluster" as described is precisely a connected component.  It is a subgraph where any two vertices are connected to each other by a path, and which is connected to no additional vertices in the rest of the graph.

The most efficient and common way to solve this is by using the Disjoint Set Union (DSU) data structure, also known as the Union-Find algorithm.

1. The DSU Data Structure

The DSU structure will keep track of which stars belong to the same cluster. It does this by assigning a representative (or parent) to each set (cluster).
parent array/map: Stores the parent of each star. Initially, every star is its own parent.

find(star_id): Finds the representative of the cluster that star_id belongs to. It does this by following the parent pointers until it reaches a node whose parent is itself (the root/representative). Path compression should be used to make this operation nearly constant time.

union_sets(starA, starB): Merges the clusters of starA and starB. It finds the representatives of both and makes one the parent of the other. Union by rank/size should be used to keep the tree flat, maintaining efficiency.

2. Algorithm Steps per Test Case

Given the constraints (up to 200 distinct stars per constellation), a simple map or a large enough array for the parent structure will work for star IDs (0 to 999). Since the problem states "Assume there are up to 200 distinct stars per constellation and all stars appearing in the input are part of a connection," we only need to manage the stars mentioned in the connections.

Step 1: Initialization
Step 2: Processing Connections (Union Operations)
Step 3: Counting Clusters

# Solution

```python
class Forest:
    def __init__(self):
        self.clusters = []

    def add(self, value):
        for cluster in self.clusters:
            # Don't re-add the value if we've already done so
            if value in cluster:
                return

        self.clusters.append(set([value]))

    # Worst case O(n), but we only have at most 100 items
    # Union-Find algorithm could make this O(log(n))
    def find(self, value):
        for k, cluster in enumerate(self.clusters):
            if value in cluster:
                return k
        return None

    # Worst case O(n), but we only have at most 100 items
    def union(self, a, b):
        k_a = self.find(a)
        k_b = self.find(b)

        if k_a == k_b:
            # already in the same group
            return

        [k1, k2] = sorted([k_a, k_b])

        # union them together
        self.clusters[k1] |= self.clusters[k2]

        # remove the second cluster since it's merged with the first
        self.clusters = self.clusters[:k2] + self.clusters[k2 + 1 :]

        # For debugging
        return self.clusters[k1]


cases = int(input())
for case in range(cases):
    f = Forest()
    connections = int(input())
    for connection in range(connections):
        a, b = map(int, input().replace("(", "").replace(")", "").split(","))
        f.add(a)
        f.add(b)
        f.union(a, b)

    print(f"Constellation #{case + 1} = {len(f.clusters)} clusters")
```