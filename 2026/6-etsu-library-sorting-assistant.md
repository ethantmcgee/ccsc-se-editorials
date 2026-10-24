# ETSU Library Sorting Assistant

## Idea

A valid call number is three pieces glued together in a fixed shape:

```
[letters][digits] . [letters][digits] <space> [4 digits]
      class            cutter               year
```

and the whole string has to be at least 10 characters and fewer than 20.

## Approach

The two delimiters are given to you, exactly one period and exactly one space. You can cut the string into its three pieces without any searching.  Split on the period to peel off the class number, then split the remainder on the space to separate the cutter from the year.

Then check each piece:

- the class number is one or more letters followed by one or more digits
- the cutter follows the same rule
- the year is exactly four digits
- the whole string falls inside the length window

A regular expression such as `^[A-Z]+[0-9]+\.[A-Z]+[0-9]+ [0-9]{4}$` does the shape in one line, and a hand-written scan over each piece works just as well.  Whichever you pick, remember the length check is a *separate* condition.

Once you have the valid call numbers, sort them with an ordinary string comparison and print one per line.

## Solutions

### Solution 1

```python
import re

cases = int(input())

def is_alpha_followed_by_number(val):
    i = 0
    has_found_number = False
    while i < len(val):
        if i == 0 and not val[i].isalpha():
            return False
        elif has_found_number and val[i].isalpha():
            return False
        elif val[i].isdigit():
            has_found_number = True
        elif not val[i].isalnum(): # not a number or digit
            return False
        i += 1
    return has_found_number

def is_all_numbers(val):
    for i in val:
        if not i.isdigit():
            return False
    return True

valid = []
for _ in range(cases):
    possible = input()
    class_num, rest = possible.split(".")
    cutter, year = rest.split(" ")
    if 10 <= len(possible) < 20 and is_alpha_followed_by_number(class_num) and is_alpha_followed_by_number(cutter) and len(year) == 4 and is_all_numbers(year):
        valid += [possible]


for x in sorted(valid):
    print(x)
```

### Solution 2

```python
import re

cases = int(input())

valid = []
for _ in range(cases):
    possible = input()
    if 10 <= len(possible) < 20 and re.match(r"^[A-Z]+[0-9]+\.[A-Z]+[0-9]+ [0-9]{4}$", possible.upper()):
        valid += [possible]

for x in sorted(valid):
    print(x)
```
