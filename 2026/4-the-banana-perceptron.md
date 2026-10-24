# The Banana Perceptron

## Idea

The whole decision is stated outright in the problem:

```
banana  <=>  yellowness >= 0.85  AND  curvature >= 0.70
```

Both conditions have to hold, so an OR won't work here, and nearly all of the difficulty is in producing exactly the output that was asked for.

## Approach

Read the fruit count, then loop.  For each fruit, read the two numbers, apply the test, print the labelled line, and add to a running count when it passes.  Once the loop finishes, print a blank line and then the total.

## Complexity

Linear in the number of fruits, which is at most 100.  There is nothing here to optimize.

## Solution

```python
cases = int(input())

bananas = 0
i = 1
for _ in range(cases):
    yellowness, curvature = map(float, input().split(" "))
    if yellowness >= 0.85 and curvature >= 0.70:
        print(f"Fruit #{i}: Banana")
        bananas += 1
    else:
        print(f"Fruit #{i}: Not a Banana")
    i += 1
print(f"\nTotal Bananas Found: {bananas}")
```
