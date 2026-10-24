# Rummikub

## Idea

The rule that makes this problem hard is the one that sounds like a throwaway line: *you may freely rearrange, split, and combine any tiles currently on the board.* That means the board you were handed is not a structure you extend.  It is just a **multiset of tiles**.  Whatever arrangement your grandmother left behind is irrelevant; the only thing that survives her turn is which tiles are on the table.  Your job is to pick a subset of your hand, throw it into that multiset, and ask a single question:

> Can these tiles be partitioned into valid groups and runs, with nothing left over?

Sample case 2 exists to make this concrete.  The board is `B1 B2 B3 B4` and your hand is `R4 G4 K9`.  If you think of the board as a fixed run you can only append to, the answer looks like 0 — nothing extends `B1 B2 B3 B4`.  The real answer is 2, because you are allowed to break `B4` off the end and rebuild the table as `B1 B2 B3` plus the group `R4 G4 B4`. 

So the problem decomposes cleanly:

1. Write a function that decides whether a multiset of tiles can be perfectly partitioned.
2. Search your hand's subsets, largest first, for one that makes the answer yes.

## Approach

### Representing the tiles

Four colors and thirteen numbers means 52 distinct tiles.  Map each one to an integer, `color * 13 + (number - 1)`, and represent any collection as a length-52 array of **counts** — not a set, because a tile may legitimately appear twice.

### Precomputing the primitive sets

Before doing anything else, build a table of every valid set, indexed by its lowest tile:

- **Groups.**  Same number, distinct colors: the four possible 3-color choices plus the single 4-color one, for each of the 13 numbers.
- **Runs.**  Same color, consecutive numbers.  Here you only need lengths **3, 4 and 5**.  Longer runs are not a separate case — a run of 6 is two runs of 3, a run of 7 is a 3 and a 4, and so on upward.  Every run of length 3 or more decomposes into pieces of size 3, 4 and 5, so those three lengths are sufficient building blocks and generating longer ones adds nothing but work.

### The partition check

Now the core routine.  Given the 52 counts, decide whether they can be split perfectly into sets from that table.

The naive version — try every set, in every order — revisits the same arrangement in every possible ordering and is hopeless.  The fix is to impose a **canonical order**: always look at the *lowest-numbered tile still in the pool*, and only consider sets that contain it.

This is safe because in any valid partition that tile belongs to exactly one set, so trying each candidate set for it covers every possibility.  And it collapses the search, because you never build the same partition twice in different orders.  It is also exactly why the primitive table is keyed by minimum tile: the lookup for "sets starting here" is already done.

```
partition(counts):
    i = lowest index with counts[i] > 0
    if there is none:              return True      # everything placed
    for each set S in TABLE[i]:
        if every tile of S is available:
            remove S from counts
            if partition(counts):  return True
            put S back
    return False
```

Memoize this on the counts tuple.  Different hand subsets produce boards that share most of their structure, so the cache earns its keep many times over within a single test case.  Clear it between test cases, or memory grows without bound across the whole input.

### Searching the hand

With the checker in hand the outer loop is simple.  You want the *maximum* number of tiles played, so count **down**:

```
for size from len(hand) down to 0:
    for each distinct subset of that size:
        if partition(board + subset):
            answer = size; stop
```

Stopping at the first success is what makes this bearable, and it is the reason to go downward rather than upward.  Deduplicate the subsets — with a sorted hand, `set(combinations(...))` collapses the identical ones a duplicate tile would otherwise produce.

Finally, if the winning size equals the whole hand, print `WINNER` rather than the number.

## Complexity

The partition check is fast in practice.  It is exponential in principle, but the canonical-lowest-tile rule plus memoization keeps it to a few hundred states on boards of this size.

The subset enumeration is what actually costs you.  With a hand of 14 there are `2^14 = 16384` subsets, and the early exit only helps when the answer is large.  The measured spread on the judge data makes the shape obvious — all of these are hands of 14 against boards of 28 to 30.

## A faster alternative

There is a way to avoid subset enumeration entirely, known as the **Den Hertog–Hulshof** algorithm.  The idea is to stop treating the tiles as a set to be partitioned and instead **sweep the values 1 through 13 in order**, deciding as you pass each value what happens to the tiles at that value.

## Solutions

### Solution 1

```python
from itertools import combinations
from functools import lru_cache

# ==========================================
# STEP 1: Define Constants and Tile Mappings
# ==========================================
# We map colors to integers to easily convert each tile string into a unique ID from 0 to 51.
COLOR_MAP = {'R': 0, 'B': 1, 'G': 2, 'K': 3}

def tile_to_id(tile_str):
    """
    Converts a tile string (e.g., 'R5', 'K13') into a unique integer index (0 to 51).
    Formula: color_index * 13 + (number - 1)
    """
    color = COLOR_MAP[tile_str[0]]
    number = int(tile_str[1:]) - 1
    return color * 13 + number

# ==========================================
# STEP 2: Precompute Valid Primitive Sets
# ==========================================
# Instead of searching for ANY valid set on the fly, we pre-generate all possible 
# foundational valid sets ("primitives").
# Optimization: We only store a set in the list of its *minimum* tile ID. 
# This guarantees we don't do redundant checks during our backtracking phase.

SETS_STARTING_WITH = [[] for _ in range(52)]

# 2a. Generate primitive GROUPS (same number, different colors)
for n in range(13):
    # Colors: 0=R, 1=B, 2=G, 3=K. We create combinations of 3 and 4 colors.
    # Group of 3 combinations:
    SETS_STARTING_WITH[0*13 + n].append((0*13+n, 1*13+n, 2*13+n))
    SETS_STARTING_WITH[0*13 + n].append((0*13+n, 1*13+n, 3*13+n))
    SETS_STARTING_WITH[0*13 + n].append((0*13+n, 2*13+n, 3*13+n))
    SETS_STARTING_WITH[1*13 + n].append((1*13+n, 2*13+n, 3*13+n))
    
    # Group of 4 combination:
    SETS_STARTING_WITH[0*13 + n].append((0*13+n, 1*13+n, 2*13+n, 3*13+n))

# 2b. Generate primitive RUNS (same color, consecutive numbers)
for c in range(4):
    for n in range(13):
        idx = c * 13 + n
        # Run of length 3
        if n + 2 <= 12:
            SETS_STARTING_WITH[idx].append((idx, idx+1, idx+2))
        # Run of length 4
        if n + 3 <= 12:
            SETS_STARTING_WITH[idx].append((idx, idx+1, idx+2, idx+3))
        # Run of length 5
        if n + 4 <= 12:
            SETS_STARTING_WITH[idx].append((idx, idx+1, idx+2, idx+3, idx+4))
            
# Note: We do NOT need to generate runs of length 6+. Why? 
# A run of 6 can be mathematically split into two valid runs of 3. 
# A run of 7 is a run of 3 and a run of 4. Therefore, lengths 3, 4, and 5 act as the 
# building blocks to form ANY valid run greater than or equal to 3.

# ==========================================
# STEP 3: The Recursive Validity Checker
# ==========================================
@lru_cache(None)
def is_valid_board(counts_tuple):
    """
    Given a tuple of 52 integers representing the counts of each tile,
    determine if all tiles can be perfectly partitioned into valid sets.
    """
    # 1. Find the FIRST available tile in the pool.
    first_idx = -1
    for i in range(52):
        if counts_tuple[i] > 0:
            first_idx = i
            break
            
    # Base Case: If the pool is entirely empty, we've successfully partitioned everything!
    if first_idx == -1:
        return True

    # 2. Branching: Try to form a valid set that STARTS with this specific tile.
    for primitive_set in SETS_STARTING_WITH[first_idx]:
        can_form = True
        
        # Check if we have all the required tiles for this primitive set
        for tile in primitive_set:
            if counts_tuple[tile] == 0:
                can_form = False
                break
                
        # If we have the tiles, "play" them and recurse to check the remainder
        if can_form:
            # Tuples are immutable (which is required for the @lru_cache memoization), 
            # so we convert it to a list to modify it, then back to a tuple.
            new_counts = list(counts_tuple)
            for tile in primitive_set:
                new_counts[tile] -= 1
                
            # If the remaining tiles form a valid board, this branch wins!
            if is_valid_board(tuple(new_counts)):
                return True
                
    # If all branches fail, this configuration is invalid.
    return False

# ==========================================
# Main Logic
# ==========================================
if __name__ == '__main__':
    t = int(input())
    for _ in range(t):
        hand_strs = input().split()
        board_strs = input().split()
        
        # Convert strings to our integer IDs
        hand_ids = [tile_to_id(t) for t in hand_strs]
        board_ids = [tile_to_id(t) for t in board_strs]
        
        # We want to maximize tiles played. We start by seeing if we can play 
        # a subset equal to the full hand (size N), checking downwards to size 0.
        solved = False
        for size in range(len(hand_ids), -1, -1):
            
            # Use a set over combinations to avoid testing identical subsets.
            # Sorting `hand_ids` ensures identical items group together.
            unique_subsets = set(combinations(sorted(hand_ids), size))
            
            for subset in unique_subsets:
                # Count frequencies of all tiles on the board + this specific hand subset
                counts = [0] * 52
                for tile in board_ids: counts[tile] += 1
                for tile in subset: counts[tile] += 1
                
                # Check if this combined board state is valid
                if is_valid_board(tuple(counts)):
                    if size == len(hand_ids):
                        print("WINNER")
                    else:
                        print(size)
                    solved = True
                    break
            
            if solved:
                break
                
        # Clear the cache between test cases to prevent memory ballooning
        is_valid_board.cache_clear()
```

### Solution 2

```python
cases = int(input())

MAX_TILES_OF_COLOR = 2

COLORS = {
    "R": 0, # red
    "G": 1, # green
    "B": 2, # blue
    "K": 3, # black
}

def groups_ok(g):
    """g = (r, b, gr, k): tiles of each colour set aside for groups at ONE value.
    Can they be split entirely into groups of 3 or 4 distinct colours?"""
    if sum(g) == 0:
        return True # nothing left over: success

    # find the first group that has a tile
    first = 0
    while g[first] == 0:
        first += 1
    # first is now effectively a color index
    # find the other groups with a tile
    others = [c for c in range(4) if c != first and g[c] > 0]
    # others are also color indexes

    # groups of 3: `first` plus any two of the other available colours
    for i in range(len(others)):
        for j in range(i + 1, len(others)):
            nxt = list(g) # clone g
            # remove group
            nxt[first] -= 1
            nxt[others[i]] -= 1
            nxt[others[j]] -= 1
            # recursively check if we can still make groups
            if groups_ok(tuple(nxt)):
                return True

    # group of 4: only possible if all three other colours are available
    if len(others) == 3:
        # remove one tile from every group
        nxt = [c - 1 for c in g]
        # recursively check if we can still make groups
        if groups_ok(tuple(nxt)):
            return True

    return False

def colour_moves(state, board, hand):
    """(open1, open2, open3) + supply -> list of (new_state, group_leftover, hand_used)."""
    runs_length_1, runs_length_2, runs_length_3plus = state
    needed_to_continue = runs_length_1 + runs_length_2
    out = []
    for plus_run_3 in range(runs_length_3plus + 1):          # optionally extend 3plus runs
        for plus_new_run in range(MAX_TILES_OF_COLOR + 1):   # start this many brand new runs
            for plus_group in range(MAX_TILES_OF_COLOR + 1): # set this many aside for groups
                # calculate how many tiles we are committing
                commit = needed_to_continue + plus_run_3 + plus_new_run + plus_group
                # do we even have that many?
                if commit < board or commit > board + hand:
                    continue
                # we assume here that we are always continuing runs of length 1 and 2
                na, nb, nc = plus_new_run, runs_length_1, runs_length_2 + plus_run_3
                out.append(((na, nb, nc), plus_group, commit - board))
    return out


for _ in range(cases):
    hand = {i: [0] * 14 for i, _ in COLORS.items()}
    board = {i: [0] * 14 for i, _ in COLORS.items()}

    hand_ct = 0
    for tile in input().split():
        hand[tile[0]][int(tile[1:])] += 1
        hand_ct += 1
    for tile in input().split():
        board[tile[0]][int(tile[1:])] += 1

    # (R, G, B, K): hand tiles played
    # each color is comprised of a tuple where
    # (# of runs of length 1 for this color, # of runs of length 2 for this color, # of runs of length 3+ for this color)
    states = {((0, 0, 0), (0, 0, 0), (0, 0, 0), (0, 0, 0)): 0}
    for i in range(1, 14):
        new_states = {}
        for st, played in states.items():
            moves_by_color = [colour_moves(st[j], board[c][i], hand[c][i]) for c, j in COLORS.items()]
            for red in moves_by_color[COLORS["R"]]:
                for green in moves_by_color[COLORS["G"]]:
                    for blue in moves_by_color[COLORS["B"]]:
                        for black in moves_by_color[COLORS["K"]]:
                            g = (red[1], green[1], blue[1], black[1])
                            if not groups_ok(g):
                                continue
                            next_state = (red[0], green[0], blue[0], black[0])
                            val = played + red[2] + green[2] + blue[2] + black[2]
                            if new_states.get(next_state, -1) < val:
                                new_states[next_state] = val
        states = new_states

    # find the best number of tiles used where there are no open runs of length 1 or 2
    best = max((v for s, v in states.items() if all(c[0] == 0 and c[1] == 0 for c in s)), default=None)
    print("WINNER" if best == hand_ct else best)
```
