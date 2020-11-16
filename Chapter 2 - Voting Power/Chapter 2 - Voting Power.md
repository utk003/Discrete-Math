# Voting Power
## Initialization
```Mathematica
quota = 8;
weights = {6, 4, 1, 2};
voterCount = Dimensions[weights][[1]];

indices = Table[x, {x, voterCount}];
```

## Bhanzaf Power Index
```Mathematica
bhanzaf = Table[0, voterCount];
```
```Mathematica
subsets = Subsets[indices];
subsetCount = Dimensions[subsets][[1]];
```
```Mathematica
For[i = 1, i <= subsetCount, i++,
  total = 0;
  For[j = 1, j <= Dimensions[subsets[[i]]][[1]], j++,
    total += weights[[subsets[[i]][[j]]]]
  ];
  If[total < quota, Continue[]];
  For[j = 1, j <= Dimensions[subsets[[i]]][[1]], j++,
    total -= weights[[subsets[[i]][[j]]]];
    bhanzaf[[subsets[[i]][[j]]]] += If[total < quota, 1, 0];
    total += weights[[subsets[[i]][[j]]]];
  ];
]
```
```Mathematica
For[i = 1; bhanzafPowerSum = 0, i <= voterCount, i++, bhanzafPowerSum += bhanzaf[[i]]]
For[i = 1, i <= voterCount, i++, bhanzaf[[i]] /= bhanzafPowerSum]
```
```Mathematica
bhanzaf
```
## Shapley-Shubik Power Index
```Mathematica
shubik = Table[0, voterCount];
```
```Mathematica
perms = Permutations[indices];
permsCount = Dimensions[perms][[1]];
```
```Mathematica
For[i = 1, i <= permsCount, i++,
  total = 0;
  For[j = 1, j <= Dimensions[perms[[i]]][[1]], j++,
    total += weights[[perms[[i]][[j]]]];
    If[total >= quota,
      shubik[[perms[[i]][[j]]]]++;
      Break[]
    ];
  ]
]
```
```Mathematica
nFactorial = voterCount!;
For[i = 1, i <= voterCount, i++, shubik[[i]] /= nFactorial]
```
```Mathematica
shubik
```
