# python
1. single int
``` python
n = int(input())
print(n)
```
2. tuple of int
``` python
n, m = map(int, input().split())
print(n, m)
```
3. list of int
``` python
n = int(input())
arr = list(map(int, input().split()))
arr.sort()
print(arr)
print(''.join(map(str, arr)))
```
4. matrix
``` python
n, m = map(int, input().split())
matrix = []
for i in range(n):
    row = list(map(int, input().split()))
    matrix.append(row)

for row in matrix:
    print(' '.join(map(str, row)))
```

5. multiple lines of input
``` python
T = int(input())
for _ in range(T):
    n = int(input())
    arr = list(map(int, input().split()))
    print(sum(arr))
```
6. string
``` python
s = input()
print(s)
print(s[::-1])  # reverse string
```
7. graph input
``` python
n, m = map(int, input().split()) # number of nodes and edges
graph = [[] for _ in range(n)]   # adjacency list
for _ in range(m): # read edges
    u, v = map(int, input().split()) # edge from u to v
    graph[u].append(v)
    graph[v].append(u)  # if undirected

for i in range(n):
    print(f"Node {i}: {graph[i]}")
```

