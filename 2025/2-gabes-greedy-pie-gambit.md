# Gabe's Greedy Pie Gambit

The problem is a longest increasing subsequence problem that can be solved in a straightforward manner with dynamic programming, with complexity n^2.

But this specific problem has a very large input size so we need to do an optimization using binary search. 

# Solution

```python
cases = int(input())
for case in range(cases):
    pie_count = int(input())
    pies = [int(x) for x in input().split(" ")][:pie_count]
    
    # tails[i] stores the smallest tail element for subsequences of length i+1
    tails = []
    
    for pie in pies:
        # Binary search for the position to insert/replace
        left, right = 0, len(tails)
        while left < right:
            mid = (left + right) // 2
            if tails[mid] < pie:
                left = mid + 1
            else:
                right = mid
        
        # If num is larger than all elements, append it
        if left == len(tails):
            tails.append(pie)
        else:
            # Replace the element at position left
            tails[left] = pie
    
    print(f"Case {case + 1}: {pie_count - len(tails)}")
```