# Networks & Trees
## Input (Degree Matrix - Adjacency Matrix)
```Mathematica
input = {
  {, "A", "B", "C", "D", "E", "F", "G", "H", "I"},
  {"A", 4, -1, 0, 0, -1, -1, -1, 0, 0},
  {"B", -1, 4, -1, -1, -1, 0, 0, 0, 0},
  {"C", 0, -1, 3, -1, 0, 0, 0, 0, -1},
  {"D", 0, -1, -1, 4, -1, 0, 0, 0, -1},
  {"E", -1, -1, 0, -1, 6, -1, 0, -1, -1},
  {"F", -1, 0, 0, 0, -1, 4, -1, -1, 0},
  {"G", -1, 0, 0, 0, 0, -1, 3, -1, 0},
  {"H", 0, 0, 0, 0, -1, -1, -1, 4, -1},
  {"I", 0, 0, -1, -1, -1, 0, 0, -1, 4}
};
input // TableForm
```

## Graph Input
### Data Input
```Mathematica
totalNumVertices = 9;
(* Doesn't need to be perfect. Just needs to identify each edge at least once each. *)
networkEdgesINPUT = {"A:BEFG", "B:CDEA", "C:DBI", "D:EI", "E:ABDFIH", "F:AEHG", "G:AFH", "H:I"};
```

### Helper Method
```Mathematica
alphabet = {"A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"};
index[letter_] := (
  pos = Position[alphabet, letter];
  If[Dimensions[pos][[1]] == 0, pos = -1, pos = pos[[1, 1]]];
  pos
)
```

### Calculations
```Mathematica
input = Table[Table[0, {c, totalNumVertices + 1}], {r, totalNumVertices + 1}];
input[[1, 1]] = Null;
For[i = 1, i <= totalNumVertices, i++,
  input[[i + 1, 1]] = input[[1, i + 1]] = alphabet[[i]];
]
For[i = Dimensions[networkEdgesINPUT][[1]], i >= 1, i--,
  split = StringSplit[networkEdgesINPUT[[i]], ":"];
  p1 = index[split[[1]]] + 1;
  split = StringSplit[split[[2]], ""];
  For[j = Dimensions[split][[1]], j >= 1, j--,
    p2 = index[split[[j]]] + 1;
    input[[p1, p2]] = input[[p2, p1]] = -1;
  ]
]
For[i = 2, i <= totalNumVertices + 1, i++,
  input[[i, i]] = -Total[Delete[input[[i]], 1]]
]
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
len = Dimensions[input][[1]];
matrix = Table[Table[input[[r, c]], {c, 2, len}], {r, 2, len}];
numSpanningTrees[matrix]
numSpanningTreesMatrix[matrix]
```
