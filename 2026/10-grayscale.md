# Grayscale

## Idea

This is a parsing problem with a one-line formula in the middle.  The formula is given to you.  The work is in reading the input correctly, and the traps are all in the input format.

So the plan is:

1. Drop comment lines.
2. Split everything else into tokens.
3. Read the four header tokens, then walk the rest three at a time.

## Approach

### Strip comments first, tokenise second

The order matters.  If you tokenise first and then try to skip the `#`, you will skip the `#` token and keep everything after it on that line: `# Sample 1: 2x2 Image` turns into the tokens `Sample`, `1:`, `2x2`, `Image`, and your parser fails when it tries to turn `Sample` into a width.  Worse, a comment like `# 1 2 3` yields tokens that *look* like pixel data and silently shift every pixel that follows.

Go line by line, throw away any line that begins with `#`, and split what is left:

```python
tokens = []
for line in sys.stdin.read().splitlines():
    if line.startswith("#"):
        continue
    tokens.extend(line.split())
```

Splitting on whitespace rather than on a single space handles runs of spaces for free — the sample input has several.

### Read the header, then the pixels

```python
w, h, m = int(tokens[1]), int(tokens[2]), int(tokens[3])
for i in range(4, 4 + 3 * w * h, 3):
    r, g, b = int(tokens[i]), int(tokens[i + 1]), int(tokens[i + 2])
    gray = (299 * r + 587 * g + 114 * b) // 1000
```

`tokens[0]` is `P3`.  Echo `P3`, `W H` and `M` back out, then print every gray value three times — `gray gray gray` — all on one line, separated by single spaces.

### Use the integer formula exactly as written

The statement gives `(299 × R + 587 × G + 114 × B) // 1000` with integer division, and gives it that way on purpose.  Two other things people reach for produce different numbers:

## Complexity

Linear in the size of the input.  There is nothing to optimise; the only performance concern is reading and writing a few megabytes efficiently, which any buffered I/O handles.

## Solutions

### Solution 1

```python
import sys

def solve():
    raw_content = sys.stdin.read()
    tokens = []
    
    # Parse tokens while ignoring PPM '#' comments
    for line in raw_content.splitlines():
        line = line.split('#')[0].strip()
        if line:
            tokens.extend(line.split())
            
    if not tokens:
        return

    magic = tokens[0]
    width = int(tokens[1])
    height = int(tokens[2])
    max_val = int(tokens[3])

    # Print PPM Header
    print("P3")
    print(f"{width} {height}")
    print(max_val)

    # Process pixel triplets
    idx = 4
    num_pixels = width * height
    out_pixels = []
    
    for _ in range(num_pixels):
        r = int(tokens[idx])
        g = int(tokens[idx+1])
        b = int(tokens[idx+2])
        idx += 3

        # Weighted Luminosity Integer Math
        gray = (299 * r + 587 * g + 114 * b) // 1000
        out_pixels.append(f"{gray} {gray} {gray}")

    # Output formatted pixel blocks
    print(" ".join(out_pixels))

if __name__ == "__main__":
    solve()
```

### Solution 2

```python
import sys

# remove all lines without a comment, and split those lines on spaces
non_comment_lines = [line.strip().split() for line in sys.stdin.readlines() if line.strip()[0] != "#"]
# flatten the list of lists into a single list removing any items with a len(0)
tokens = [x for xs in non_comment_lines for x in xs if len(x) > 0]

assert tokens[0] == "P3"
print("P3")

w, h, m = tokens[1], tokens[2], tokens[3]
print(f"{w} {h}")
print(m)

first = True
for i in range(4, len(tokens), 3):
    r, g, b = map(int, [tokens[i], tokens[i + 1], tokens[i + 2]])
    gray = (299 * r + 587 * g + 114 * b) // 1000
    if first:
        print(f"{gray} {gray} {gray}", end="")
        first = False
    else:
        print(f" {gray} {gray} {gray}", end="")
print()
```
