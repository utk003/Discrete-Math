# Networks & Trees
## Input (Degree Matrix - Adjacency Matrix)
```Mathematica
input = {
  {, "A", "B", "C", "D", "E", "F", "G"},
  {"A", 2, -1, -1, 0, 0, 0, 0},
  {"B", -1, 3, -1, -1, 0, 0, 0},
  {"C", -1, -1, 3, -1, 0, 0, 0},
  {"D", 0, -1, -1, 3, -1, 0, 0},
  {"E", 0, 0, 0, -1, 3, -1, -1},
  {"F", 0, 0, 0, 0, -1, 2, -1},
  {"G", 0, 0, 0, 0, -1, -1, 2}
};
input // TableForm
```

## Helper Methods (Do NOT Modify)
```Mathematica
completeMatrix[n_] := (
  table = Table[Table[-1, {k, n}], {k, n}];
  For[i = 1, i <= n, i++, table[[i, i]] = n - 1];
  table
)
```
```Mathematica
cofactor[m_, i_, j_] := (
  l1 = Dimensions[m][[1]];
  l2 = Dimensions[m][[2]];
  row = Delete[Table[k, {k, l1}], i];
  col = Delete[Table[k, {k, l2}], j];
  Table[Table[m[[r, c]], {c, col}], {r, row}]
)
```
```Mathematica
numSpanningTrees[m_] := Det[cofactor[m, 1, 1]]
numSpanningTreesMatrix[m_] := (
  l1 = Dimensions[m][[1]];
  l2 = Dimensions[m][[2]];
  Table[Table[(-1)^(r + c) Det[cofactor[m, r, c]], {c, l2}], {r, l1}]
)
```

## Output (# of spanning trees for network)
```Mathematica
numSpanningTrees[matrix]
```
