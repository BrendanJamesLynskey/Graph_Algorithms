# Graph Algorithms

**Computer Science Fundamentals Series**

BFS · DFS · Topological sort · Connected components · Cycle detection · Graph coloring

*Mid-level software engineer track -- 21 slides*

---

## Table of Contents

1. [Graph Representations -- Adjacency Matrix](#slide-02--graph-representations--adjacency-matrix)
2. [Adjacency List & Edge List](#slide-03--adjacency-list--edge-list)
3. [Breadth-First Search (BFS)](#slide-04--breadth-first-search-bfs)
4. [Depth-First Search (DFS)](#slide-05--depth-first-search-dfs)
5. [BFS vs DFS -- When to Use Which](#slide-06--bfs-vs-dfs--when-to-use-which)
6. [Topological Sorting](#slide-07--topological-sorting)
7. [Connected Components](#slide-08--connected-components)
8. [Strongly Connected Components](#slide-09--strongly-connected-components)
9. [Cycle Detection](#slide-10--cycle-detection)
10. [Bipartite Checking](#slide-11--bipartite-checking)
11. [Articulation Points & Bridges](#slide-12--articulation-points--bridges)
12. [Euler Paths & Circuits](#slide-13--euler-paths--circuits)
13. [Hamiltonian Paths & Circuits](#slide-14--hamiltonian-paths--circuits)
14. [Graph Coloring](#slide-15--graph-coloring)
15. [Planarity](#slide-16--planarity)
16. [Directed Acyclic Graphs (DAGs)](#slide-17--directed-acyclic-graphs-dags)
17. [Graph Algorithm Complexity Reference](#slide-18--graph-algorithm-complexity-reference)
18. [Common Patterns & Techniques](#slide-19--common-patterns--techniques)
19. [Real-World Applications](#slide-20--real-world-applications)
20. [Summary & Further Reading](#slide-21--summary--further-reading)

---

## Slide 02 -- Graph Representations -- Adjacency Matrix

A graph `G = (V, E)` can be stored as a 2D matrix `M[i][j]` where entry `(i, j)` is 1 (or the edge weight) if an edge exists between vertices `i` and `j`, and 0 otherwise.

| Property | Value |
|----------|-------|
| Space | `O(V²)` |
| Edge lookup | `O(1)` |
| Iterate neighbours | `O(V)` |
| Add edge | `O(1)` |
| Remove edge | `O(1)` |

> **Best for:** dense graphs where `|E| ≈ |V|²`, or when constant-time edge lookup is critical (e.g. Floyd-Warshall).

---

## Slide 03 -- Adjacency List & Edge List

### Adjacency list

Each vertex stores a list of its neighbours. The most common representation for sparse graphs.

```python
graph = {
    0: [1, 3],
    1: [0, 2],
    2: [1, 3],
    3: [0, 2]
}
```

| Property | Value |
|----------|-------|
| Space | `O(V + E)` |
| Edge lookup | `O(degree)` |
| Iterate neighbours | `O(degree)` |
| Add edge | `O(1)` |

### Edge list

A flat list of `(u, v)` pairs (optionally with weight). Minimal storage; useful for algorithms like Kruskal's that iterate all edges.

```python
edges = [
    (0, 1), (0, 3),
    (1, 2), (2, 3)
]
```

> **Rule of thumb:** use an adjacency list unless you have a specific reason not to. It offers the best balance of space and query performance for most graph problems.

---

## Slide 04 -- Breadth-First Search (BFS)

Explore all vertices at distance `d` before any at distance `d+1`. Uses a **queue** (FIFO).

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []
    while queue:
        v = queue.popleft()
        order.append(v)
        for nb in graph[v]:
            if nb not in visited:
                visited.add(nb)
                queue.append(nb)
    return order
```

| Property | Value |
|----------|-------|
| Time | `O(V + E)` |
| Space | `O(V)` |
| Data structure | Queue (FIFO) |

### Applications

- Shortest path in unweighted graphs
- Level-order traversal
- Connected components in undirected graphs
- Bipartite checking
- Web crawlers, social-network degree separation

> BFS guarantees the **shortest path** (fewest edges) from source to every reachable vertex in an unweighted graph.

---

## Slide 05 -- Depth-First Search (DFS)

Explore as deep as possible along each branch before backtracking. Uses a **stack** (LIFO) -- or recursion.

```python
def dfs(graph, start):
    visited = set()
    order = []
    stack = [start]
    while stack:
        v = stack.pop()
        if v in visited:
            continue
        visited.add(v)
        order.append(v)
        for nb in reversed(graph[v]):
            if nb not in visited:
                stack.append(nb)
    return order
```

### Recursive variant

```python
def dfs_rec(graph, v, visited=None):
    if visited is None:
        visited = set()
    visited.add(v)
    for nb in graph[v]:
        if nb not in visited:
            dfs_rec(graph, nb, visited)
    return visited
```

| Property | Value |
|----------|-------|
| Time | `O(V + E)` |
| Space | `O(V)` |
| Data structure | Stack (LIFO) / recursion |

### Applications

- Topological sorting
- Cycle detection
- Strongly connected components
- Maze solving & puzzle search
- Articulation points and bridges

### Edge classification (directed graphs)

| Edge type | Description |
|-----------|------------|
| **Tree edge** | Edge in the DFS tree itself |
| **Back edge** | Points to an ancestor -- indicates a cycle |
| **Forward edge** | Points to a descendant (non-tree) |
| **Cross edge** | Points to a non-ancestor, non-descendant |

---

## Slide 06 -- BFS vs DFS -- When to Use Which

| Criterion | BFS | DFS |
|-----------|-----|-----|
| Strategy | Level by level (queue) | Branch by branch (stack) |
| Shortest path (unweighted) | **Yes** | No |
| Memory on wide graphs | High -- stores entire frontier | **Low** -- single branch |
| Memory on deep graphs | **Low** | High -- deep stack |
| Cycle detection (directed) | Possible but awkward | **Natural** via back edges |
| Topological sort | Kahn's algorithm (BFS) | Reverse post-order (DFS) |
| Connected components | Yes | Yes |
| Articulation points / bridges | Not used | **Standard approach** |

> **BFS rule:** when you need the shortest path or must explore by distance (level-order), choose BFS.

> **DFS rule:** when you need to explore all paths, detect cycles, or compute post-order (topological sort, SCC), choose DFS.

---

## Slide 07 -- Topological Sorting

A linear ordering of vertices in a **DAG** (directed acyclic graph) such that for every directed edge `u → v`, vertex `u` comes before `v`.

### Kahn's algorithm (BFS)

```python
from collections import deque

def kahn(graph, n):
    indeg = [0] * n
    for u in range(n):
        for v in graph[u]:
            indeg[v] += 1
    queue = deque(v for v in range(n)
                    if indeg[v] == 0)
    order = []
    while queue:
        u = queue.popleft()
        order.append(u)
        for v in graph[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                queue.append(v)
    if len(order) != n:
        raise ValueError("Cycle detected")
    return order
```

### DFS-based approach

```python
def topo_dfs(graph, n):
    visited = [False] * n
    stack = []
    def dfs(u):
        visited[u] = True
        for v in graph[u]:
            if not visited[v]:
                dfs(v)
        stack.append(u)
    for v in range(n):
        if not visited[v]:
            dfs(v)
    return stack[::-1]
```

### Applications

- Build system dependency resolution (Make, Gradle)
- Course prerequisite scheduling
- Task scheduling with precedence constraints
- Spreadsheet cell evaluation order
- Package manager install ordering

> A topological ordering exists **if and only if** the graph is a DAG. Both Kahn's and DFS run in `O(V + E)`.

---

## Slide 08 -- Connected Components

### Undirected graphs

A connected component is a maximal set of vertices such that there is a path between every pair. Finding all components: run BFS/DFS from each unvisited vertex.

```python
def connected_components(graph, n):
    visited = [False] * n
    components = []
    for v in range(n):
        if not visited[v]:
            comp = []
            stack = [v]
            while stack:
                u = stack.pop()
                if visited[u]:
                    continue
                visited[u] = True
                comp.append(u)
                for nb in graph[u]:
                    if not visited[nb]:
                        stack.append(nb)
            components.append(comp)
    return components
```

Time: `O(V + E)` -- Space: `O(V)`

### Union-Find alternative

Disjoint Set Union (DSU) with path compression and union by rank achieves near-constant time per operation.

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(
                self.parent[x])
        return self.parent[x]
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py: return
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
```

> Union-Find is `O(α(n))` per operation -- effectively constant. Preferred for dynamic connectivity queries and Kruskal's MST.

---

## Slide 09 -- Strongly Connected Components

In a **directed graph**, a strongly connected component (SCC) is a maximal set of vertices where every vertex is reachable from every other vertex in the set.

### Kosaraju's algorithm

1. Run DFS on the original graph; record finish order
2. Transpose the graph (reverse all edges)
3. Run DFS on the transpose in reverse finish order
4. Each DFS tree in step 3 is one SCC

```python
def kosaraju(graph, n):
    # Step 1: finish order via DFS
    visited, order = [False]*n, []
    def dfs1(u):
        visited[u] = True
        for v in graph[u]:
            if not visited[v]: dfs1(v)
        order.append(u)
    for v in range(n):
        if not visited[v]: dfs1(v)
    # Step 2: transpose
    rev = [[] for _ in range(n)]
    for u in range(n):
        for v in graph[u]: rev[v].append(u)
    # Step 3: DFS on transpose
    visited = [False]*n
    sccs = []
    def dfs2(u, comp):
        visited[u] = True
        comp.append(u)
        for v in rev[u]:
            if not visited[v]: dfs2(v, comp)
    for u in reversed(order):
        if not visited[u]:
            c = []
            dfs2(u, c)
            sccs.append(c)
    return sccs
```

### Tarjan's algorithm

Single-pass DFS using a stack and low-link values. More efficient in practice -- no need to transpose the graph.

```python
def tarjan(graph, n):
    idx = [0]
    stack, on_stack = [], [False]*n
    index = [-1]*n
    lowlink = [0]*n
    sccs = []
    def strongconnect(u):
        index[u] = lowlink[u] = idx[0]
        idx[0] += 1
        stack.append(u)
        on_stack[u] = True
        for v in graph[u]:
            if index[v] == -1:
                strongconnect(v)
                lowlink[u] = min(
                    lowlink[u], lowlink[v])
            elif on_stack[v]:
                lowlink[u] = min(
                    lowlink[u], index[v])
        if lowlink[u] == index[u]:
            comp = []
            while True:
                w = stack.pop()
                on_stack[w] = False
                comp.append(w)
                if w == u: break
            sccs.append(comp)
    for v in range(n):
        if index[v] == -1:
            strongconnect(v)
    return sccs
```

> Both algorithms run in `O(V + E)`. Tarjan uses one DFS pass; Kosaraju uses two but is conceptually simpler.

---

## Slide 10 -- Cycle Detection

### Directed graphs -- DFS colouring

Track three states: WHITE (unvisited), GREY (in current DFS path), BLACK (fully processed). A back edge to a GREY vertex indicates a cycle.

```python
def has_cycle_directed(graph, n):
    WHITE, GREY, BLACK = 0, 1, 2
    colour = [WHITE] * n

    def dfs(u):
        colour[u] = GREY
        for v in graph[u]:
            if colour[v] == GREY:
                return True   # back edge
            if colour[v] == WHITE:
                if dfs(v):
                    return True
        colour[u] = BLACK
        return False

    return any(colour[v] == WHITE
               and dfs(v)
               for v in range(n))
```

### Undirected graphs

Simpler: during DFS, if a neighbour is already visited and is **not** the parent, there is a cycle.

```python
def has_cycle_undirected(graph, n):
    visited = [False] * n

    def dfs(u, parent):
        visited[u] = True
        for v in graph[u]:
            if not visited[v]:
                if dfs(v, u):
                    return True
            elif v != parent:
                return True
        return False

    return any(not visited[v]
               and dfs(v, -1)
               for v in range(n))
```

> **Kahn's shortcut:** if topological sort processes fewer than `V` vertices, the directed graph contains a cycle.

---

## Slide 11 -- Bipartite Checking

A graph is **bipartite** if its vertices can be coloured with two colours such that no edge connects two vertices of the same colour. Equivalently: the graph contains no odd-length cycle.

### BFS 2-colouring

```python
from collections import deque

def is_bipartite(graph, n):
    colour = [-1] * n
    for start in range(n):
        if colour[start] != -1:
            continue
        colour[start] = 0
        queue = deque([start])
        while queue:
            u = queue.popleft()
            for v in graph[u]:
                if colour[v] == -1:
                    colour[v] = 1 - colour[u]
                    queue.append(v)
                elif colour[v] == colour[u]:
                    return False
    return True
```

Time: `O(V + E)` -- same as a single BFS/DFS pass.

### Applications

- Job assignment problems (workers ↔ tasks)
- Graph colouring with 2 colours
- Stable matching (Gale-Shapley)
- Conflict-free scheduling
- Network flow modelling

> **König's theorem:** in a bipartite graph, the size of the maximum matching equals the size of the minimum vertex cover.

---

## Slide 12 -- Articulation Points & Bridges

### Definitions

An **articulation point** (cut vertex) is a vertex whose removal disconnects the graph. A **bridge** (cut edge) is an edge whose removal disconnects the graph.

### Tarjan's bridge-finding

```python
def find_bridges(graph, n):
    disc = [-1] * n
    low = [-1] * n
    timer = [0]
    bridges = []
    def dfs(u, parent):
        disc[u] = low[u] = timer[0]
        timer[0] += 1
        for v in graph[u]:
            if disc[v] == -1:
                dfs(v, u)
                low[u] = min(low[u], low[v])
                if low[v] > disc[u]:
                    bridges.append((u, v))
            elif v != parent:
                low[u] = min(low[u], disc[v])
    for v in range(n):
        if disc[v] == -1:
            dfs(v, -1)
    return bridges
```

### Key insight: low-link values

For each vertex `u`, `low[u]` is the smallest discovery time reachable from the subtree rooted at `u` (including back edges).

- **Bridge:** edge `(u, v)` is a bridge if `low[v] > disc[u]`
- **Cut vertex:** non-root `u` is a cut vertex if some child `v` has `low[v] >= disc[u]`. Root is a cut vertex if it has >= 2 DFS children.

### Applications

- Network reliability analysis -- single points of failure
- Critical infrastructure in communication networks
- Biconnected component decomposition
- Router/link failure planning in ISP networks

> Both algorithms run in `O(V + E)`. Removing a bridge always increases the number of connected components by exactly one.

---

## Slide 13 -- Euler Paths & Circuits

An **Euler path** visits every *edge* exactly once. An **Euler circuit** is an Euler path that starts and ends at the same vertex.

### Existence conditions

|  | Euler Path | Euler Circuit |
|--|-----------|--------------|
| **Undirected** | Exactly 0 or 2 odd-degree vertices | All vertices have even degree |
| **Directed** | At most one vertex with `out - in = 1`, at most one with `in - out = 1`, rest equal | Every vertex has `in-degree = out-degree` |

### Hierholzer's algorithm

```python
def euler_circuit(graph, n):
    # graph: adjacency list (mutable)
    adj = [list(nbrs) for nbrs in graph]
    stack = [0]
    circuit = []
    while stack:
        v = stack[-1]
        if adj[v]:
            u = adj[v].pop()
            # For undirected, also remove
            # reverse edge from adj[u]
            stack.append(u)
        else:
            circuit.append(stack.pop())
    return circuit[::-1]
```

Time: `O(E)` -- each edge is processed exactly once.

> **Königsberg bridges (1736):** Euler proved no walk can cross all seven bridges exactly once -- the birth of graph theory.

### Applications

- DNA fragment assembly (de Bruijn graphs)
- Chinese postman problem
- Circuit board routing
- Network routing optimisation

---

## Slide 14 -- Hamiltonian Paths & Circuits

A **Hamiltonian path** visits every *vertex* exactly once. A **Hamiltonian cycle** is a Hamiltonian path that returns to the start vertex.

### Euler vs Hamiltonian

|  | Euler | Hamiltonian |
|--|-------|-------------|
| Visits | Every **edge** once | Every **vertex** once |
| Existence check | `O(V + E)` -- degree conditions | **NP-complete** |
| Finding path | `O(E)` | Exponential (backtracking) |

### Backtracking approach

```python
def hamiltonian_path(graph, n):
    path = [0]
    visited = {0}
    def backtrack():
        if len(path) == n:
            return True
        u = path[-1]
        for v in graph[u]:
            if v not in visited:
                visited.add(v)
                path.append(v)
                if backtrack():
                    return True
                path.pop()
                visited.remove(v)
        return False
    if backtrack():
        return path
    return None
```

### Sufficient conditions

- **Dirac:** if every vertex has degree >= `n/2`, a Hamiltonian cycle exists
- **Ore:** if `deg(u) + deg(v) >= n` for every non-adjacent pair, a Hamiltonian cycle exists

> **NP-complete:** no known polynomial-time algorithm exists for deciding whether a Hamiltonian path/cycle exists. This is one of the classic hard problems in CS.

---

## Slide 15 -- Graph Coloring

Assign colours to vertices so that no two adjacent vertices share the same colour. The minimum number of colours needed is the **chromatic number** `χ(G)`.

### Greedy colouring

```python
def greedy_colour(graph, n):
    colour = [-1] * n
    for u in range(n):
        used = {colour[v] for v in graph[u]
                if colour[v] != -1}
        c = 0
        while c in used:
            c += 1
        colour[u] = c
    return colour
```

Greedy uses at most `Δ + 1` colours, where `Δ` is the maximum degree. Not always optimal.

### Key chromatic numbers

| Graph type | χ(G) |
|-----------|------|
| Bipartite | 2 |
| Odd cycle | 3 |
| Complete graph `K_n` | `n` |
| Planar graph | ≤ 4 (four colour theorem) |
| Tree | 2 (if ≥ 2 vertices) |

### Applications

- Register allocation in compilers (interference graph)
- Exam timetabling -- no student has two exams at once
- Map colouring
- Frequency assignment in cellular networks
- Sudoku (9-colouring of the constraint graph)

> **Brooks' theorem:** every connected graph (not a complete graph or odd cycle) can be coloured with at most `Δ` colours.

> Deciding if `χ(G) ≤ k` is NP-complete for `k ≥ 3`. For `k = 2` it reduces to bipartite checking -- solvable in `O(V + E)`.

---

## Slide 16 -- Planarity

A graph is **planar** if it can be drawn on a plane without any edge crossings.

### Euler's formula for planar graphs

`V - E + F = 2`

where `V` = vertices, `E` = edges, `F` = faces (including the outer face).

### Corollaries

- For a simple planar graph: `E ≤ 3V - 6`
- For a bipartite planar graph: `E ≤ 2V - 4`
- Every planar graph has a vertex of degree ≤ 5

### Non-planar graphs

- **K₅** -- complete graph on 5 vertices. Has 10 edges but `3(5) - 6 = 9` -- violates the bound.
- **K₃,₃** -- complete bipartite graph. Has 9 edges but `2(6) - 4 = 8` -- violates the bipartite bound.

> **Kuratowski's theorem:** a graph is planar if and only if it contains no subdivision of K₅ or K₃,₃.

### Testing planarity

- **Hopcroft-Tarjan:** linear-time `O(V)` planarity test based on path addition
- **Boyer-Myrvold:** linear-time algorithm that also produces the planar embedding or a Kuratowski subgraph

### Applications

- Circuit board layout (PCB routing)
- VLSI chip design
- Geographic information systems
- The four colour theorem applies only to planar graphs

---

## Slide 17 -- Directed Acyclic Graphs (DAGs)

A DAG is a directed graph with no directed cycles. DAGs are the foundation for dependency modelling and scheduling.

### Properties

- Has at least one topological ordering
- Has at least one source (in-degree 0) and one sink (out-degree 0)
- Longest path is computable in `O(V + E)` via topological sort + dynamic programming
- Number of paths between two vertices computable in `O(V + E)`

### Longest path in a DAG

```python
def longest_path_dag(graph, n):
    order = topo_dfs(graph, n)
    dist = [0] * n
    for u in order:
        for v in graph[u]:
            dist[v] = max(dist[v],
                          dist[u] + 1)
    return max(dist)
```

### DAG applications

- **Build systems** -- Make, Bazel, Gradle model task dependencies as DAGs
- **Version control** -- Git commit history forms a DAG
- **Data pipelines** -- Apache Airflow, dbt model ETL stages as DAGs
- **Neural networks** -- computation graphs in deep learning are DAGs

> **Critical path:** the longest path in a project DAG determines the minimum completion time. Shortening any non-critical task does not reduce total time.

---

## Slide 18 -- Graph Algorithm Complexity Reference

| Algorithm | Time | Space | Use case |
|-----------|------|-------|----------|
| BFS / DFS | `O(V + E)` | `O(V)` | Traversal, shortest path (unweighted), components |
| Topological sort | `O(V + E)` | `O(V)` | Dependency ordering in DAGs |
| Kosaraju / Tarjan SCC | `O(V + E)` | `O(V)` | Strongly connected components |
| Bridges / articulation points | `O(V + E)` | `O(V)` | Network reliability, critical links |
| Union-Find | `O(α(n))` per op | `O(V)` | Dynamic connectivity, Kruskal's MST |
| Dijkstra (binary heap) | `O((V+E) log V)` | `O(V)` | Shortest path (non-negative weights) |
| Bellman-Ford | `O(VE)` | `O(V)` | Shortest path (handles negative weights) |
| Floyd-Warshall | `O(V³)` | `O(V²)` | All-pairs shortest paths |
| Greedy colouring | `O(V + E)` | `O(V)` | Approximate graph colouring |
| Hierholzer (Euler) | `O(E)` | `O(E)` | Euler path/circuit |
| Hamiltonian path | Exponential | `O(V)` | NP-complete -- backtracking or DP bitmask |
| Planarity test | `O(V)` | `O(V)` | Hopcroft-Tarjan, Boyer-Myrvold |

> **Pattern:** most fundamental graph algorithms run in `O(V + E)`. Exponential complexity is a red flag that the problem may be NP-hard -- look for special structure (DAG, tree, planar) to unlock polynomial solutions.

---

## Slide 19 -- Common Patterns & Techniques

### Multi-source BFS

Start BFS from multiple sources simultaneously. Used for rotten oranges, nearest 0 in a grid, fire spread simulation.

```python
queue = deque(all_sources)
# standard BFS from here
```

### 0-1 BFS

When edge weights are only 0 or 1, use a deque: push 0-weight neighbours to front, 1-weight to back. Runs in `O(V + E)`.

```python
if weight == 0:
    dq.appendleft(v)
else:
    dq.append(v)
```

### Implicit graphs

The graph is never materialised -- neighbours are generated on demand. Puzzles, game states, word ladders.

### Bidirectional BFS

Run BFS from both source and target simultaneously. Meets in the middle -- reduces search space from `O(b^d)` to `O(b^(d/2))`.

### Graph as grid

2D grids are implicit graphs. Each cell connects to 4 (or 8) neighbours. Flood fill, island counting, shortest path in a maze.

### Bitmask DP on graphs

For small `n` (≤ 20), represent visited sets as bitmasks. Travelling salesman: `O(2^n · n²)` DP.

> **Interview tip:** most graph problems reduce to BFS/DFS + a twist. Identify the vertices, edges, and what "visited" means -- the algorithm often writes itself.

---

## Slide 20 -- Real-World Applications

| Domain | Graph model | Algorithms used |
|--------|------------|----------------|
| Social networks | Users = vertices, friendships = edges | BFS (degrees of separation), SCC, community detection |
| Navigation / maps | Intersections = vertices, roads = weighted edges | Dijkstra, A*, contraction hierarchies |
| Compilers | Interference graph (variables sharing lifetimes) | Graph colouring for register allocation |
| Package managers | Packages = vertices, dependencies = directed edges | Topological sort, cycle detection |
| Network infrastructure | Routers = vertices, links = edges | Bridges, articulation points, MST |
| Bioinformatics | DNA k-mers = edges in de Bruijn graph | Euler paths for genome assembly |
| Fraud detection | Accounts = vertices, transactions = edges | Cycle detection, SCC, anomaly scoring |
| Scheduling | Tasks = vertices, precedence = directed edges | Topological sort, critical path (DAG longest path) |
| Web crawling | Pages = vertices, hyperlinks = directed edges | BFS, PageRank (random walk on graph) |
| Garbage collection | Objects = vertices, references = edges | DFS reachability from root set |

> **Graphs are everywhere.** Any time you have entities with pairwise relationships -- whether explicit (friend lists) or implicit (grid cells) -- you are working with a graph.

---

## Slide 21 -- Summary & Further Reading

### Key takeaways

- Adjacency lists are the default representation -- `O(V + E)` space, efficient neighbour iteration
- BFS finds shortest paths in unweighted graphs; DFS is the workhorse for cycle detection, topological sort, and SCC
- Topological sort only works on DAGs and runs in `O(V + E)`
- Tarjan's and Kosaraju's find SCCs in linear time
- Bridges and articulation points identify single points of failure in networks
- Graph colouring is NP-hard for `k ≥ 3` but greedy heuristics work well in practice
- Euler paths are tractable (`O(E)`); Hamiltonian paths are NP-complete
- Most interview graph problems reduce to BFS or DFS with a creative state definition

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- the canonical algorithms textbook; chapters 20-26 cover graphs comprehensively |
| **Sedgewick** | *Algorithms, 4th ed.* -- excellent graph algorithm visualisations and Java implementations |
| **Kleinberg & Tardos** | *Algorithm Design* -- network flow, matching, and advanced graph topics with clear motivation |
| **CP-Algorithms** | [cp-algorithms.com](https://cp-algorithms.com) -- competitive programming reference with proofs and implementations |
| **MIT 6.006** | Erik Demaine's Introduction to Algorithms -- free on MIT OCW |
