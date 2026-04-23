# Keyboard Key Distance Matrix Methodology

## Objective

The goal of this method is to construct a **distance matrix between keyboard keys** based on their physical neighborhood on a keyboard layout.

This matrix allows us to measure how far apart two keys are in terms of **movement across adjacent keys**, rather than using raw Cartesian coordinates. In other words, the distance between two keys is defined as the **minimum number of key-to-key steps** needed to move from one key to another on the keyboard graph.

This is useful when we want to model:

- hand movement effort
- local typing transitions
- likely finger travel patterns
- structural similarity between typed character sequences

---

## Input Data

The method uses two main inputs.

### 1. Key-to-code mapping table

A CSV file `cap_eng_keycodes.csv` is read to map each key name to a numeric key code.

Example concept:

| Key | KeyCode |
|---|---:|
| A | 65 |
| S | 83 |
| D | 68 |

This mapping is needed because the matrices are indexed by numeric key code.

### 2. Keyboard layout grid

The keyboard is represented as a 2D grid:

```python
key_grid = [
    ['Grave Accent', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', 'Dash', 'Equal Sign', 'Backspace'],
    ['Tab', 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P', 'Open Bracket', 'Close Bracket', 'Back Slash'],
    ['Caps Lock', 'A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', 'Semi-colon', 'Single Quote', 'Enter'],
    ['Shift', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', 'Comma', 'Period', 'Forward Slash']
]
```

This grid is a simplified structural representation of the keyboard. Each element corresponds to one physical key position.

---

## Step 1: Constructing the Keyboard Graph

The keyboard is modeled as a **graph**:

- each key is a **node**
- an edge is added between two keys if they are **adjacent on the keyboard**

Adjacency includes all 8 surrounding directions:

- up
- down
- left
- right
- upper-left
- upper-right
- lower-left
- lower-right

This is defined by:

```python
directions = [
    (-1, -1), (-1, 0), (-1, 1),
    (0, -1),           (0, 1),
    (1, -1),  (1, 0),  (1, 1)
]
```

### Interpretation

If key `U` is directly beside key `V` in any of these directions, then the graph connects them with distance 1.

So this graph does **not** use exact geometric measurements in centimeters or pixel coordinates. Instead, it uses **topological neighborhood distance**.

---

## Step 2: Building the Adjacency Matrix

An adjacency matrix `adj_mat` of size `300 x 300` is created:

```python
adj_mat = [[0] * 300 for _ in range(300)]
```

### Why size 300?

This creates enough index space to store all possible key codes used in the mapping. Only the positions corresponding to actual key codes are used.

### How entries are filled

For each key in the keyboard grid:

1. find its key code
2. check all 8 neighboring positions
3. if the neighbor exists and also has a valid key code
4. mark the connection in the adjacency matrix

Formally:

- `adj_mat[i][j] = 1` if key `i` and key `j` are adjacent
- `adj_mat[i][j] = 0` otherwise

### Meaning of the adjacency matrix

The adjacency matrix represents the **direct one-step movement structure** of the keyboard.

For example:

- `adj_mat[A][S] = 1` because `A` and `S` are next to each other
- `adj_mat[A][D] = 0` if they are not immediate neighbors
- diagonal values are still initially `0`

The adjacency matrix is then saved as:

```python
with open("adj_mat.pkl", "wb") as f:
    pickle.dump(adj_mat, f)
```

---

## Step 3: Initializing the Distance Matrix

Next, a distance matrix `dist_mat` is created:

```python
dist_mat = [[float('inf')] * 300 for _ in range(300)]
```

This matrix will eventually store the **shortest path distance** between every pair of keys.

### Initialization rule

For each pair `(i, j)`:

- if `i == j`, then distance is `0`
- if `adj_mat[i][j] == 1`, then distance is `1`
- otherwise, distance is initialized to infinity

This is implemented as:

```python
if adj_mat[i][j] == 1 or i == j:
    dist_mat[i][j] = adj_mat[i][j]
```

Because:

- when `i == j`, `adj_mat[i][j] = 0`, so self-distance becomes `0`
- when two keys are adjacent, distance becomes `1`

### Meaning

At this stage:

- the matrix knows all direct neighbor distances
- it does **not yet know** multi-step distances such as moving from `A` to `P`

---

## Step 4: Computing All-Pairs Shortest Paths

To compute the minimum number of steps between every pair of keys, the algorithm uses **Floyd-Warshall**:

```python
for k in range(300):
    for i in range(300):
        for j in range(300):
            dist_mat[i][j] = min(dist_mat[i][j], dist_mat[i][k] + dist_mat[k][j])
```

### What Floyd-Warshall does

For each possible intermediate key `k`, the algorithm checks whether going from `i` to `j` through `k` gives a shorter path than the current known path.

This gradually updates the matrix until every entry contains the **shortest graph distance**.

### Result

After this step:

- `dist_mat[i][j] = minimum number of adjacent-key moves from key i to key j`

Examples conceptually:

- same key to itself → `0`
- neighboring keys → `1`
- keys separated by one intermediate key → `2`
- distant keys across the board → larger values

The final matrix is saved as:

```python
with open("dist_mat.pkl", "wb") as f:
    pickle.dump(dist_mat, f)
```

---

## Graph Interpretation of the Distance

This methodology defines keyboard distance as:

> the minimum number of adjacency hops required to travel from one key to another on the keyboard graph

This is a **graph-based movement distance**, not Euclidean distance.

### Why this is useful

This representation is often better than raw coordinate distance when studying typing behavior because:

- it reflects **reachable neighborhood structure**
- it is robust to irregular row lengths
- it naturally handles non-rectangular keyboard layouts
- it captures how keys are locally connected during typing

---

## Example

Suppose we want the distance from `A` to `D`.

- `A` is adjacent to `S`
- `S` is adjacent to `D`

So the shortest path may be:

`A -> S -> D`

Thus:

- `dist(A, D) = 2`

If two keys touch diagonally, they still count as adjacent, so their distance is `1`.

---

## Assumptions

This method makes several simplifying assumptions.

### 1. All adjacent moves have equal cost

Moving left, right, up, down, or diagonally all cost `1`.

So diagonal movement is treated the same as horizontal movement.

### 2. Keyboard is represented structurally, not physically

The layout is based on row and column neighborhood, not exact physical spacing.

### 3. Only listed keys are considered

Keys not included in the mapping or layout are ignored.

### 4. Static keyboard layout

The method assumes a fixed keyboard geometry.

---

## Strengths

This methodology has several advantages:

- simple and easy to compute
- interpretable
- captures local keyboard structure well
- works naturally with graph-based or time-series feature engineering
- suitable for comparing key transitions in behavioral typing data

---

## Limitations

There are also limitations.

### 1. No true physical spacing

Some keys on a real keyboard may be physically wider or offset, but that is not modeled exactly.

### 2. Diagonal movement may be oversimplified

In reality, diagonal movement may not have the same ergonomic cost as horizontal movement.

### 3. Finger assignment is not modeled

The matrix measures key-to-key structural distance, not which finger is likely used.

### 4. Modifier and language behavior are excluded

This method focuses only on key positions and adjacency, not Shift usage, language switching, or actual keystroke mechanics.

---

## Time Complexity

### Adjacency matrix construction

This is efficient because each key checks only 8 neighbors.

### Floyd-Warshall

The all-pairs shortest path computation has complexity:

\[
O(N^3)
\]

where `N = 300`.

This is acceptable here because the matrix size is small and fixed.

---

## Summary

The distance matrix is created in four main stages:

1. **Map keys to numeric key codes**
2. **Represent the keyboard as a 2D grid**
3. **Build an adjacency matrix using 8-direction neighborhood**
4. **Run Floyd-Warshall to compute shortest path distances between all keys**

The final result is a matrix where each entry gives the **minimum adjacency-hop distance** between two keys on the keyboard.

This gives a clean and interpretable way to quantify keyboard movement structure for downstream analysis such as typing dynamics, transition patterns, clustering, or anomaly detection.

---

## Pseudocode Summary

```text
Load key-to-code mapping from CSV
Define keyboard layout as a 2D grid

Initialize adjacency matrix A

For each key position (r, c):
    For each of 8 neighbor directions:
        If neighbor exists and both keys have valid codes:
            Set A[code_u][code_v] = 1

Initialize distance matrix D:
    D[i][i] = 0
    D[i][j] = 1 if A[i][j] = 1
    D[i][j] = inf otherwise

Run Floyd-Warshall:
    For each intermediate node k:
        For each source node i:
            For each target node j:
                D[i][j] = min(D[i][j], D[i][k] + D[k][j])

Save adjacency matrix and distance matrix
```
