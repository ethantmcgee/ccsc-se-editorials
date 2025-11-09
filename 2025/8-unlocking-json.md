# Unlocking JSON

JSON is a data transfer format that is composed of dictionaries (maps or objects), lists, strings, integers, and booleans.  These can come in any combination.  For example, all of the following are valid JSON.

```json
0
true
"hello"
["hello", "world"]
{"hello": "world"}
```

The entry in a list or value in a dictionary can be any valid JSON type.  From the problem description, we can quickly gather that all of our JSON codexes will be objects at the top most level, but thereafter, we must be somewhat creative.  We can see that we are only going to have to deal with integers are strings since we can't really index with a boolean, and we are guaranteed from the problem statement that object properties will be strings.

Let's solve this problem in Python since it has a nice json processing library.  This library allows us to easily convert the json codex into a dictionary.

```python
import json

dictionary = json.loads(str)
```

From there, we simply need to break apart our input strings.  We can see that `.` and `[` are our property accessors dividers.  We can use the python regex library to separate on both at the same time.

```python
import re

cases = int(input())

for case in range(cases):
    access_path = re.split('[\\[\\.]', input())
```

At this point, we can see that our accessor is now either string or number, but it could end with a pesky `]` that we'll need to remove.

# Solution

```python
import json
import re

# the codex comes after the input, so we store the accessors until the codex is read
accessors = []
cases = int(input())
for case in range(cases):
    # read the accessor and split on [ and .
    parts = re.split('[\\[\\.]', input())
    # strip the pesky ]
    parts = [part.rstrip(']') for part in parts]
    accessors.append(parts)

# just read until we crash to build the codex
codex = ""
done = False
while not done:
    try:
        codex += input()
    except:
        done = True

# convert the codex to a dictionary
codex = json.loads(codex)

# now we iterate over over each accessor
for accessor in accessors:
    # copy the codex root
    start = codex
    for part in accessor:
        try: # see if integer access works
            start = start[int(part)]
        except: # nope probably a string
            start = start[part]
    print(start, end='')
```
