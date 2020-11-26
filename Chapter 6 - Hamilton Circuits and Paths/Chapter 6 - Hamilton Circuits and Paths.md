# Hamilton Circuits and Paths
## Input
### Data Table (distances)
```Mathematica
input = {
  {, "A", "B", "C", "D", "E", "F", "G", "H", "I", "J"},
  {"A", \[Infinity], 185, 119, 152, 133, 321, 297, 277, 412, 381},
  {"B", 185, \[Infinity], 121, 150, 200, 404, 458, 492, 379, 427},
  {"C", 119, 121, \[Infinity], 174, 120, 332, 439, 348, 245, 443},
  {"D", 152, 150, 174, \[Infinity], 199, 495, 480, 500, 454, 489},
  {"E", 133, 200, 120, 199, \[Infinity], 315, 463, 204, 396, 487},
  {"F", 321, 404, 332, 495, 315, \[Infinity], 356, 211, 369, 222},
  {"G", 297, 458, 439, 480, 463, 356, \[Infinity], 471, 241, 235},
  {"H", 277, 492, 348, 500, 204, 211, 471, \[Infinity], 283, 478},
  {"I", 412, 379, 245, 454, 396, 369, 241, 283, \[Infinity], 304},
  {"J", 381, 427, 443, 489, 487, 222, 235, 478, 304, \[Infinity]}
};
input // TableForm
```

## Nearest Neighbor/Repeated Nearest Neighbor Algorithms (Do NOT Modify)
### NNA Search (Do NOT Modify)
```Mathematica
runNNASearch[inputArr_, posMap_, startIndex_] := (
  posMapLen = Dimensions[posMap][[1]];
  indices = posMap;
  indices[[startIndex]] = 0;
  curr = start[[startIndex]];
  temp = {curr};
  td = 0;
  For[size = 2, size <= posMapLen, size++,
    min = -1;
    next = -1;
    For[ind = 1, ind <= posMapLen, ind++,
      If[indices[[ind]] > 0,
        If[min == -1 || min > inputArr[[curr, indices[[ind]]]],
          min = inputArr[[curr, indices[[ind]]]];
          next = ind;
        ];
      ];
    ];
    curr = indices[[next]];
    temp = Append[temp, curr];
    td += min;
    indices[[next]] = 0;
  ];
  td += inputArr[[temp[[1]], temp[[posMapLen]]]];
  {td, temp}
)
```

### RNNA Search (Do NOT Modify)
```Mathematica
result = {};
dist = -1;
len = Dimensions[input][[1]] - 1;
start = Table[i + 1, {i, 1, len}];
For[i = 1, i <= len, i++,
  res = runNNASearch[input, start, i];
  If[dist == -1 || dist > res[[1]],
    dist = res[[1]];
    result = res[[2]];
  ];
]
Append[Table[input[[1, result[[i]]]], {i, 1, len}], input[[1, result[[1]]]]]
dist
```

## Cheapest Link (Do NOT Modify)
### Helper Methods (Do NOT Modify)
```Mathematica
cheapestLink[posMap_, weights_] := (
  weightsCopy = weights;
  posMapLen = Dimensions[posMap][[1]];
  min = \[Infinity];
  For[r = 1, r <= posMapLen, r++,
    For[c = 1, c <= posMapLen, c++,
      val = weightsCopy[[posMap[[r]], posMap[[c]]]];
      If[val >= 0 && val < min, min = val]
    ];
  ];
  ans = {1, 1};
  For[r = 1, r <= posMapLen, r++,
    For[c = 1, c <= posMapLen, c++,
      If[weightsCopy[[posMap[[r]], posMap[[c]]]] == min,
        ans = {r, c};
        Break[];
      ];
    ];
  ];
  p1 = posMap[[ans[[1]]]];
  p2 = posMap[[ans[[2]]]];
  weightsCopy[[p1, p2]] = weightsCopy[[p2, p1]] = \[Infinity];
  {ans, weightsCopy}
)

clearVertex[posMap_, weights_, p1_, p2_] := (
  weightsCopy = weights;
  posMapLen = Dimensions[posMap][[1]];
  weightsSize = Dimensions[weightsCopy][[1]];
  If[p1 == p2 || p1 > posMapLen || p2 > posMapLen || p1 < 1 || p2 < 1 || weightsCopy[[posMap[[p1]], posMap[[p2]]]] != \[Infinity],
    Return[weightsCopy]
  ];
  For[i$clearVertex = 2, i$clearVertex <= weightsSize, i$clearVertex++,
    If[i$clearVertex != posMap[[p1]] && weightsCopy[[posMap[[p1]], i$clearVertex]] == \[Infinity],
      weightsCopy[[i$clearVertex, posMap[[p2]]]] = weightsCopy[[posMap[[p2]], i$clearVertex]] = \[Infinity];
    ];
    If[i$clearVertex != posMap[[p2]] && weightsCopy[[posMap[[p2]], i$clearVertex]] == \[Infinity],
      weightsCopy[[i$clearVertex, posMap[[p1]]]] = weightsCopy[[posMap[[p1]], i$clearVertex]] = \[Infinity];
    ];
  ];
  weightsCopy
)

clearVertices[posMap_, weights_] := (
  tempWeights = weights;
  numVerts = Dimensions[input][[1]] - 1;
  For[i$clearVertices = 1, i$clearVertices <= numVerts, i$clearVertices++,
    For[j$clearVertices = 1, j$clearVertices <= numVerts, j$clearVertices++,
      tempWeights = clearVertex[posMap, tempWeights, i$clearVertices, j$clearVertices];
    ];
  ];
  tempWeights
)

fixRows[weights_, vertDegrees_] := (
  copyWeights = weights;
  For[i = 1, i <= numVerts, i++,
    If[vertDegrees[[i]] >= 2,
      For[i2 = 1, i2 <= numVerts, i2++,
        If[copyWeights[[i + 1, i2 + 1]] != \[Infinity],
          copyWeights[[i + 1, i2 + 1]] = copyWeights[[i2 + 1, i + 1]] = -\[Infinity]
        ]
      ]
    ];
  ];
  copyWeights
)

constructPath[edges_] := (
  copyEdges = edges;
  res = {1};
  len = Dimensions[copyEdges][[1]];
  curr = 1;
  While[len > 0,
    For[i = 1, i <= len, i++,
      If[copyEdges[[i, 1]] == curr || copyEdges[[i, 2]] == curr, Break[]]
    ];
    curr = Total[copyEdges[[i]]] - curr;
    res = Append[res, curr];
    copyEdges = Delete[copyEdges, i];
    len--;
  ];
  res + 1
)
```

### Cheapest Link Search (Do NOT Modify)
```Mathematica
numVertices = Dimensions[input][[1]] - 1;
pMap = Table[k + 1, {k, numVertices}];
costs = input;
len = 1;
path = {};
degrees = Table[0, {k, numVertices}];
While[len < numVertices,
  result = cheapestLink[pMap, costs];
  path = Append[path, result[[1]]];
  degrees[[result[[1, 1]]]]++;
  degrees[[result[[1, 2]]]]++;
  costs = fixRows[clearVertices[pMap, result[[2]]], degrees];
  len++;
]
last = {};
For[i = 1, i < numVertices, i++, If[degrees[[i]] != 2, last = Append[last, i]]]
path = Append[path, last];
cost = 0;
For[i = 1, i <= numVertices, i++, cost += input[[path[[i, 1]] + 1, path[[i, 2]] + 1]]]
path = constructPath[path];
Table[input[[1, path[[i]]]], {i, 1, numVertices + 1}]
cost
```

## Brute Force Search
```Mathematica
len = Dimensions[input][[1]] - 1;
start = Table[i + 1, {i, 2, len}];
start = Permutations[start];
min = -1;
max = 0;
p1 = {};
p2 = {};
For[i = Dimensions[start][[1]], i > 0, i--,
  curr = Prepend[start[[i]], 2];
  d = input[[curr[[1]], curr[[len]]]];
  For[i2 = 1, i2 < len, i2++, d += input[[curr[[i2]], curr[[i2 + 1]]]]];
  If[min == -1 || min > d,
    min = d;
    p1 = curr;
  ];
  If[max < d,
    max = d;
    p2 = curr;
  ];
]
Append[Table[input[[1, p1[[i]]]], {i, 1, len}], input[[1, p1[[1]]]]]
min
Append[Table[input[[1, p2[[i]]]], {i, 1, len}], input[[1, p2[[1]]]]]
max
```
