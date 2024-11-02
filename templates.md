## Binsearch

```python
def binsearch(i, j, v):
    if (i == j):
        return i if numbers[i] == v else None
    m = (i + j) // 2
    if (numbers[m] == v): return m
    if (numbers[m] < v):
        return binsearch(m+1, j, v)
    if (numbers[m] > v):
        return binsearch(i, m-1, v)
```

