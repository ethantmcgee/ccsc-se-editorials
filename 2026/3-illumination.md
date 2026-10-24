# Illumination

## Idea

This problem requires the solver to recognize which space to search.  It can be quite tempting to search the button subspace, but with up to 50 buttons that is 2^50 subsets, not possible within the time limits at the upper bounds of the problem.

Instead, search the light configurations.  With at most 8 lights there are at most 2^8 = 256 of them, quite reasonable.

What connects the two is XOR.  Encode a configuration as a number with one bit per light, and encode each button the same way, with a 1 in every position that button toggles.  Pressing a button is then exactly:

```
new_configuration = configuration ^ button
```

That gives you three useful facts for free:

- Order does not matter, because XOR is commutative.
- Pressing the same button twice does nothing, because XOR is its own inverse.
- So any sequence of presses is really just a *subset* of buttons, and the answer you are looking for is a subset size.

## Approach

Breadth-first search over configurations.  Start from the previous configuration with a press count of 0.  For each configuration you have reached, XOR it with every button in turn.  If the result is one you have not seen, record it with a press count one higher and add it to the list to expand next.  If you have seen it before, throw it away — you already reached it in no more presses than this.  Keep going until nothing new turns up.

Because BFS expands configurations in order of press count, the *first* time you reach a configuration is already the fewest presses that can reach it.  When the search runs dry, look up the desired configuration: if it is in your table, print its count, and if it was never reached, print `Impossible`.

## Complexity

At most 256 configurations, each tried against at most 50 buttons, is roughly 12,800 steps.

## Solution

```python
def to_number(listy):
    i = 0
    for x in listy:
        i <<= 1
        if x == 1:
            i |= 1
        else:
            i |= 0
    return i

lights, buttons, cases = map(int, input().split(" "))
button_set = []
for _ in range(buttons):
    button_set.append(to_number(map(int, input().split(" "))))

for _ in range(cases):
    start = to_number(map(int, input().split(" ")))
    end = to_number(map(int, input().split(" ")))

    found = {start: 0}
    to_try = [start]
    presses = 1
    while len(to_try) > 0:
        new_to_try = []
        for conf in to_try:
            for button in button_set:
                if conf ^ button not in found:
                    found[conf ^ button] = presses
                    new_to_try += [conf ^ button]
        presses += 1
        to_try = new_to_try

    if end in found:
        print(found[end])
    else:
        print("Impossible")
```
