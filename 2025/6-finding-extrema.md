# Finding Extrema

This is a fun problem that combines Calculus theory (the Extreme Value Theorem) with a numerical approach for finding extrema on a closed interval for the function f(x) below. 

The program implements the exact sampling method described in the problem: checking f(x) at a, b, and s-1 equally spaced steps between them (a total of s+1 points) to numerically approximate the minimum and maximum values.  

```python
from functools import cache

cases = int(input())

@cache
def f(i):
    return i**3 - 5*(i**2) + 5*i + 2

for case in range(cases):
    min_val, min_pos, max_val, max_pos = None, None, None, None
    min, max, steps = [float(x) for x in input().split(" ")]

    increment = (max - min) / steps
    for i in range(int(steps) + 1):
        val = f(min + increment * i)
        if min_val is None or val < min_val:
            min_val = val
            min_pos = min + increment * i
        if max_val is None or val > max_val:
            max_val = val
            max_pos = min + increment * i

    print(f"case {case + 1}, minimum of {round(min_val, 1)} at x={round(min_pos, 1)}, maximum of {round(max_val, 1)} at x={round(max_pos, 1)}")

```