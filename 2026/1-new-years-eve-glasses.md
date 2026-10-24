# New Year's Eve Glasses

## Idea

This problem revolves around the fact that there are only certain digits through which a wearer can see:
```
visible = ["0", "4", "6", "8", "9"]
```

Solving the problem means taking a year and determining if the two middle digits both fall into this list.  If they do, a simple print can be done.  However, if they cannot, it is time to begin an upward search for the next year that works.

## Approach

Check the second and third characters of the year.  If both are visible, print the `Goodbye <year - 1>, hello <year>!` form.  Otherwise, increment the year and check again, repeating until you land on a wearable year, and then print that year on its own.

Working with the year as a string is convenient here, since the digits you care about are just positions 1 and 2.

## Complexity

The upward search is short.  The worst gap is exactly 300 years: at 1100 the second digit is a 1, and the next visible digit in that position is a 4, so the search has to walk all the way to 1400.  The same thing happens at 2100, 3100, and so on up the range.  With at most 1000 test cases, that is 300,000 checks in the very worst case.

The search is also guaranteed to terminate inside the allowed range. 9999 is wearable!

## Solution

```python
visible = ["0", "4", "6", "8", "9"]

def is_visible(year):
    return year[1] in visible and year[2] in visible

cases = int(input())
for _ in range(cases):
    year = input()
    if is_visible(year):
        print(f"Goodbye {int(year) - 1}, hello {year}!")
    else:
        while not is_visible(year):
            year = str(int(year) + 1)
        print(year)
```
