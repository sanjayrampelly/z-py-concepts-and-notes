# Attended Interview Questions
## 1. Tuple Mutability
```python
t = ("apple", "orange", "banana")

# print(t[2]) 
# t[0] = "grapes"  # This would raise an error

print(t)

l = list(t)
l[2] = "grapes"

t = tuple(l)
print(t)
```

### 🧠 Key Difference

**❌ Not Allowed:**
```python
t[0] = "grapes"
```
Trying to modify an element inside an immutable tuple → not allowed

**✔️ Allowed:**
```python
t = tuple(l)
```
Creating a new tuple and assigning it to variable `t` → allowed

**Important:** Tuples are immutable, but variables pointing to tuples are not immutable. You can always make a new tuple and reassign the variable.

### Tuples with Mutable Elements

```python
t = ("apple", {1: "a", 2: "b"})
```

The second element is a dictionary, and dictionaries are mutable.

However, this will raise an error:
```python
t[1] = {3: "c"}  # TypeError: 'tuple' object does not support item assignment
```

**What you can do:** Modify the dictionary stored inside the tuple:
```python
t = ("apple", {1: "a", 2: "b"})
t[1][3] = "c"  # modify the dictionary
print(t)
# Output: ('apple', {1: 'a', 2: 'b', 3: 'c'})
```

This works because:
- The tuple's structure remains unchanged (it still has the same two elements)
- The dictionary object inside the tuple is mutable, so its contents can change

---

## 2. Length of List with Trailing Comma

```python
l = [2, 3.9, 4, 5,]
```

### ✅ Length of the list

The list contains **4 elements**: `2`, `3.9`, `4`, `5`

The trailing comma does not create an extra element. It just makes the list formatting easier and is totally valid in Python.

```python
len(l)  # output -> 4
```

⭐ **Length = 4**

---

## 3. Separate Uppercase and Lowercase Characters

**Problem:** `s = "AbCadbB"` → output should be `res = "ACBbadb"`

### Solution 1: Two Loops
```python
s = "AbCadbB"
res = ""

for i in s:
    if i.isupper():
        res += i

for i in s:
    if i.islower():
        res += i

print(res)
```

### ✅ Single Loop Approach
```python
s = "AbCadbB"
res_upper = ""
res_lower = ""

for ch in s:
    if ch.isupper():
        res_upper += ch
    else:
        res_lower += ch

res = res_upper + res_lower
print(res)
```

### ✅ List Comprehension Approach
```python
s = "AbCadbB"
upper_chars = [c for c in s if c.isupper()]
lower_chars = [c for c in s if c.islower()]
res = "".join(upper_chars + lower_chars)
print(res)
```

---

## 4. Interleave Two Strings

```python
s1 = "hello"
s2 = "door"
res = ""

minl = min(len(s1), len(s2))

for i in range(minl):
    res += s1[i] + s2[i]

for i in range(minl, len(s1)):
    res += s1[i]

for j in range(minl, len(s2)):
    res += s2[j]

print(res)
```

### ✅ Optimized Version
```python
s1 = "hello"
s2 = "door"
res = ""

minl = min(len(s1), len(s2))

for i in range(minl):
    res += s1[i] + s2[i]

res += s1[minl:] + s2[minl:]

print(res)
```

---

## 5. Merge Dictionaries and Sum Values

```python
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 20, "c": 30, "d": 40}

for k, v in d2.items():
    if k in d1:
        d1[k] += v
    else:
        d1[k] = v

print(d1)
```

### Alternative: Using Merge Operator
```python
merged = d1 | d2
print(merged)
```

### Sub-Question: Sort Keys by Values (Descending)
```python
print(dict(sorted(d1.items(), key=lambda x: x[1], reverse=True)))
```

### Handling Type Errors
```python
d1 = {"a": 1, "b": 2, "c": "3"}
d2 = {"b": 20, "c": 30, "d": 40}

def merge(d1, d2):
    for k, v in d2.items():
        try:
            if k in d1:
                d1[k] += v
            else:
                d1[k] = v
        except TypeError as e:
            print(e)
    
    return d1

print(merge(d1, d2))
```

### ⭐ Final Correct Rule
- ✅ If you raise inside `try` → exception will be caught → program continues
- ❌ If you raise inside `except` → new exception goes out → program stops (unless another try catches it)

---

## 6. Group Anagrams

```python
strs = ["eat", "tea", "tan", "ate", "nat", "bat"]

def grpanag(strs):
    d = {}
    for i in strs:
        key = tuple(sorted(i))
        if key in d:
            d[key].append(i)
        else:
            d[key] = [i]
    
    return d

d1 = grpanag(strs)
print(d1)

l = []
for k, v in d1.items():
    l.append(v)

print(l)
```

---

## 7. Reverse Words in a String

**Problem:** `s = "I Love Python"` → `res = "Python Love I"`

### Solution 1: Manual Parsing
```python
s = "I Love Python"
word = ""
words = []

for i in s:
    if i == " ":
        words.append(word)
        word = ""
    else:
        word += i

words.append(word)

print(words)

res = ""
for i in range(len(words) - 1, -1, -1):
    res += words[i] + " "

print(res)
```

### ✅ Optimized Version
```python
s = "I Love Python"
words = s.split()  # ['I', 'Love', 'Python']
res = " ".join(words[::-1])
print(res)
```

---

## 8. Print Unique Characters

```python
s = "doollyy"
```

### Solution 1: Using count()
```python
s = "doollyy"
res = ""
for i in s:
    if s.count(i) == 1:
        res += i

print(res)
```

### Solution 2: Using Dictionary
```python
d = {}
for i in s:
    d[i] = d.get(i, 0) + 1

for i in s:
    if d[i] == 1:
        print(i)
```

---

## 9. Find Missing Numbers

### One Number Missing
```python
l = [1, 2, 4, 6, 8]
res = []

for i in range(1, len(l)):
    if l[i] - l[i-1] != 1:
        res.append(l[i] - 1)

print(res)
```

### Multiple Numbers Missing
```python
l = [1, 2, 5, 7, 10]
res = []

for i in range(1, len(l)):
    if l[i] - l[i-1] != 1:
        res.extend(range(l[i-1] + 1, l[i]))

print(res)
```

---

## 10. Flatten a Nested List

### Recursive Approach
```python
lst = [[1, 2, 3], [4, 5], [6, 7, 8], 10]

def flatlist(lst):
    res = []
    for i in lst:
        if isinstance(i, list):
            res.extend(flatlist(i))
        else:
            res.append(i)
    
    return res

print(flatlist(lst))
```

### Iterative Approach
```python
def flatlist_iter(lst):
    res = []
    stack = lst[::-1]  # reverse to process left-to-right using pop

    while stack:
        item = stack.pop()
        if isinstance(item, list):
            stack.extend(item[::-1])  # reverse sublist before extending
        else:
            res.append(item)
    
    return res

# Example
lst = [1, [2, [3, 4], 5], [6, 7], 8]
print(flatlist_iter(lst))
```

---

## 11. Two Sum Problem

```python
arr = [2, 4, 6, -3, -1, 0, 1, -2]
target = 2
seen = set()
res = []

for i in range(len(arr)):
    diff = target - arr[i]
    
    if diff in seen:
        res.append([arr[i], diff])
    
    seen.add(arr[i])

print(res)
```

### With Index Tracking
```python
seen = {}
res = []

for i in range(len(arr)):
    diff = target - arr[i]
    if diff in seen:
        res.append([seen[diff], i])
    seen[arr[i]] = i  # store value -> index

print(res)
```

---

## 12. Flatten Nested Dictionary

```python
def flatten(d, parent_key='', sep='_'):
    items = {}

    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k

        if isinstance(v, dict):
            items.update(flatten(v, new_key, sep))

        elif isinstance(v, list):
            for i, item in enumerate(v):
                list_key = f"{new_key}_{i}"
                items[list_key] = item

        else:
            items[new_key] = v

    return items

data = {
    'id': '123',
    'name': 'Laptop',
    'specs': {
        'ram': '16GB',
        'cpu': 'i7',
        'storage': {
            'type': 'SSD',
            'size': '1TB'
        }
    },
    'tags': ['fast', 'portable', 'light']
}

print(flatten(data))
```

---

## Topics to Review

### API Optimization
- How do you optimize APIs?
- How do you optimize at API Layer?
- If an API is for both browser and app, how do you handle it and what changes are needed?

### Advanced Topics
- Merge dict questions (Sanjay ChatGPT)
- Idempotency in APIs

### SQL
- Joins
- RANK
- DENSE_RANK
- ROW_NUMBER
- (ChatGPT reference)

### Framework Differences
- How are Django and FastAPI different?
- Django and DRF revision (ChatGPT)
- FastAPI basics
