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

## Nearest Neighbor/Repeated Nearest Neighbor Algorithms
### NNA Search
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

### RNNA Search
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
Table[input[[1, result[[i]]]], {i, 1, len}]
dist
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
Table[input[[1, p1[[i]]]], {i, 1, len}]
min
Table[input[[1, p2[[i]]]], {i, 1, len}]
max
```
