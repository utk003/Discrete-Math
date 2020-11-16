# Apportionment
## Possible Methods of Apportionment (Do NOT Modify)
```Mathematica
jefferson[x_] := Floor[x]

adams[x_] := Ceiling[x]

webster[x_] := Round[x]

geoMean[x_] := Sqrt[x (x + 1)]
huntingtonHill[x_] := If[x > geoMean[Floor[x]], Ceiling[x], Floor[x]]
```

## Necessary Search Parameters
### Population/Quota Input
```Mathematica
(* State census data import *)
data = Drop[Drop[Import["USA Population Data.xlsx"][[1]], 9], -7];
data = Drop[Table[data[[k]], {k, 1, 51}], {9}];
states = Table[data[[i]][[1]], {i, 1, 50}];
census = Table[Table[data[[i]][[j]], {i, 1, 50}], {j, 2, 13}];
```
```Mathematica
apportionmentRounder[x_] := Map[huntingtonHill, x]
populations = census[[12]];
seatsToGive = 435;
```

### Necessary Metadata Calculations (Do NOT Modify)
```Mathematica
numPopulationGroups = Dimensions[populations][[1]];
totalPopulation = Total[populations];
populationPerSeat = totalPopulation/seatsToGive;
```

## Search Parameters/Function (Do NOT Modify)
### Binary Search Parameters/Variables (Do NOT Modify)
```Mathematica
variableQuotas = Table[0, {k, 1, numPopulationGroups}];
binarySearchChange = 10^(Ceiling[Log[10, populationPerSeat]] - 5);
lower = 0;
upper = populationPerSeat*2;
loopCondition = upper > lower;
```

### Binary Search (Do NOT Modify)
```Mathematica
While[loopCondition,
  variablePopulationSeatRatio = (lower + upper)/2;
  variableQuotas = apportionmentRounder[populations/variablePopulationSeatRatio];
  totalQuotas = Total[variableQuotas];
  If[totalQuotas == seatsToGive,
    loopCondition = False,
    If[totalQuotas > seatsToGive,
      lower = variablePopulationSeatRatio + binarySearchChange,
      upper = variablePopulationSeatRatio - binarySearchChange
    ];
    loopCondition = upper > lower;
  ]
]
```

### Search Output (Do NOT Modify)
```Mathematica
If[totalQuotas != seatsToGive, Print["Search was inconclusive"],
  Print["Distributions Ratio: ", variablePopulationSeatRatio // N];
  Print["Distribution Quotas: ", variableQuotas];
]
```
```Mathematica
For[i = 1, i <= 50, i++,
  Print["Number of seats for ", states[[i]], ": ", variableQuotas[[i]]];
]
```
