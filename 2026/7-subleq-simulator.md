# SUBLEQ Simulator

## Idea

This is a specification problem: build the machine exactly as described, and get the error handling right.

One instruction, `subleq(a, b, c)`:

```
mem[b] = mem[b] - mem[a]
if mem[b] <= 0:  jump to cell c
else:            continue with the next instruction (pc + 3)
```

That really is the whole processor.

## Approach

Load everything into a single flat array.  The data section fills cells 0 through `d - 1`, and the program section follows immediately after it.  Execution begins at cell `d`, the first instruction, and the machine halts when it jumps to cell 0.  Total memory is `d + 3p` cells, and anything beyond that does not exist.

While you load, note which cells are uninitialized: any whose value has magnitude 1000 or more.  Keeping their addresses in a set is enough.

Then run the loop.  For each instruction, in this order:

1. **Uninitialized operands.**  If `a` or `b` names a cell you marked, that is an error — *unless* `a` and `b` are the same cell.  `subleq(x, x, c)` is the idiom for zeroing `x`, and it never reads a meaningful value out of it, so it is allowed and it clears the mark.
2. **Out of range.**  If `a` or `b` points past the end of memory, that is the same error.  Memory beyond the program section is uninitialized by definition.
3. **Program modification.**  If `b` lands at or past the start of the program section, you are about to write into your own code.  Note the subtlety: subtracting a zero changes nothing, so it only counts as modification when `mem[a]` is non-zero.
4. **Do the subtraction.**
5. **Branch.**  If the result is `<= 0`, look at `c`.  Cell 0 is the halt: print `mem[0]` and stop.  Any other destination below the program section, or past the end of memory, is a non-standard exit.  Otherwise jump there.  If the result is positive you fall through to `pc + 3`, and running off the bottom of the program section that way is also a non-standard exit.

The order matters: stop and print at the first error you hit, without executing any further.

## Complexity

Constant work per instruction, so the cost is simply however many instructions the program retires.

## Solutions

### Solution 1

```python
# SUBLEQ Simulator Task

def process_commands(mem:list[int],start:int,uninit:set):
    # Process the commands
    pos = start
    while pos >= start:
        a,b,c = mem[pos+0], mem[pos+1], mem[pos+2]
        if a in uninit or b in uninit:
            if a != b: return "Used uninitialized memory"
            uninit.remove(a)
        if a >= len(mem) or b >= len(mem): return "Used uninitialized memory"
        if b >= start and mem[a] != 0:
            return "Program modification detected"
        mem[b] -= mem[a]
        if mem[b] <= 0:
            if c < start or c >= len(mem):
                return mem[0] if 0 == c else "Non-standard exiting of the program section"
            pos = c
        else: 
            pos += 3
            if pos >= len(mem):
                return "Non-standard exiting of the program section"
    print(mem[0])

uninit_threshold = 999
data_bnd,num_cmds = [int(v) for v in input().split()]
mem_bnd = data_bnd+3*num_cmds
mem = [0 for _ in range(mem_bnd)]
uninit = set()
for d in range(data_bnd):
    mem[d] = int(input())
    if mem[d] < -uninit_threshold or mem[d] > uninit_threshold: 
        uninit.add(d)

for p in range(num_cmds):
    vals = [int(v) for v in input().split()]
    mem[data_bnd+3*p+0] = vals[0]
    mem[data_bnd+3*p+1] = vals[1]
    mem[data_bnd+3*p+2] = vals[2]
    if vals[0] < -uninit_threshold or vals[0] > uninit_threshold:
        uninit.add(vals[0])

print(process_commands(mem,data_bnd,uninit))
```

### Solution 2

```python
d, p = map(int, input().split())

memory = []
uninit = set()
for i in range(d):
    memory += [int(input())]
    if abs(memory[i]) >= 1000:
        uninit.add(i)
for _ in range(p):
    memory += list(map(int, input().split()))

pc = d
while pc != 0 and pc < len(memory):
    a, b, c = memory[pc], memory[pc + 1], memory[pc + 2]
    if a >= len(memory) or b >= len(memory):
        print("Used uninitialized memory")
        exit()
    if a in uninit or b in uninit:
        if a != b:
            print("Used uninitialized memory"); exit()
        uninit.discard(a)
    res = memory[b] - memory[a]
    if b >= d and memory[a] != 0:
        print("Program modification detected")
        exit()
    memory[b] = res
    if res <= 0:
        if c != 0 and c < d:
              print("Non-standard exiting of the program section")
              exit()
        pc = c
    else:
        pc += 3

if pc >= len(memory):
    print("Non-standard exiting of the program section")
else:
    print(memory[0])
```
