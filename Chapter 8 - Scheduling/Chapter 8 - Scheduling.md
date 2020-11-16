# Scheduling
## Data Input
### Data Entry
```Mathematica
input = {
  {"AP", 7},
  {"AF", 5},
  {"AW", 6},
  {"AD", 8},
  {"IF", 5},
  {"IW", 7},
  {"ID", 5},
  {"PL", 4},
  {"IP", 4},
  {"PU", 3},
  {"HU", 4},
  {"IC", 1},
  {"FW", 6},
  {"PD", 3},
  {"EU", 2}
};

dependencies = {
  {},
  {},
  {},
  {},
  {"AP", "AF"},
  {"IF", "AW"},
  {"AD", "IW"},
  {"IF"},
  {"IW"},
  {"IP", "ID"},
  {"IP"},
  {"PL", "HU"},
  {"IC"},
  {"HU"},
  {"PU", "HU"}
};

numTeams = 2;
```

## Misc. Helper Methods (Do NOT Modify)
```Mathematica
loopedDependenciesChecker[] := (
  fails$loopedDependenciesChecker = False;
  For[i$loopedDependenciesChecker = 1, i$loopedDependenciesChecker <= numTasks, i$loopedDependenciesChecker++,
    For[j$loopedDependenciesChecker = i$loopedDependenciesChecker, j$loopedDependenciesChecker <= numTasks, j$loopedDependenciesChecker++,
      If[ContainsAll[dependencies[[j$loopedDependenciesChecker]], {tasks[[i$loopedDependenciesChecker]]}] && ContainsAll[dependencies[[i$loopedDependenciesChecker]], {tasks[[j$loopedDependenciesChecker]]}],
        fails$loopedDependenciesChecker = True
      ]
    ]
  ];
  fails$loopedDependenciesChecker
)
```
```Mathematica
index[task_, tasks_] := (
  ind$index = -1;
  taskLen$index = Dimensions[tasks][[1]];
  For[i$index = 1, i$index <= taskLen$index, i$index++,
    If[tasks[[i$index]] == task, ind$index = i$index]
  ];
  ind$index
)
index[task_] := index[task, tasks]
```

### Data Processing (Do NOT Modify)
```Mathematica
numTasks = Dimensions[input][[1]];

tasks = Table[input[[k, 1]], {k, numTasks}];
times = Table[input[[k, 2]], {k, numTasks}];

If[loopedDependenciesChecker[],
  dependencies = Table[{}, {k, numTasks}];
  Print["Dependencies NOT loaded (Given dependencies had cycles)"],
  Print["Dependencies successfully loaded"]
]
```
```Mathematica
dependencyMatrix = Table[Table[False, {j, numTasks}], {i, numTasks}];
For[i = 1, i <= numTasks, i++,
  dependenciesLength = Dimensions[dependencies[[i]]][[1]];
  For[j = 1, j <= dependenciesLength, j++, dependencyMatrix[[i, index[dependencies[[i, j]]]]] = True]
]
```

## Decreasing-Time Algorithm
```Mathematica
addValidTasksFromList[fromList_, dependencyMat_, toList_] := (
  fromList$addValidTasksFromList = fromList;
  toList$addValidTasksFromList = toList;
  numAvailableTasks$addValidTasksFromList = Dimensions[fromList][[1]];

  For[i$addValidTasksFromList = numAvailableTasks$addValidTasksFromList, i$addValidTasksFromList >= 1, i$addValidTasksFromList--,
    If[ContainsOnly[dependencyMat[[index[fromList$addValidTasksFromList[[i$addValidTasksFromList]]]]], {False}],
     toList$addValidTasksFromList = Append[toList$addValidTasksFromList, fromList$addValidTasksFromList[[i$addValidTasksFromList]]];
     fromList$addValidTasksFromList = Delete[fromList$addValidTasksFromList, i$addValidTasksFromList];
    ];
  ];
  {fromList$addValidTasksFromList, toList$addValidTasksFromList}
)

inputSortedDecreasing = Sort[input, #1[[2]] > #2[[2]] &];
inputSortedDecreasing = Table[inputSortedDecreasing[[i, 1]], {i, numTasks}];
giveKSlowestTasks[validTasks_, k_] := (
  If[Dimensions[validTasks][[1]] <= k,
    validTasks,
    kSlowest$giveKSlowestTasks = {};
    count$giveKSlowestTasks = 0;
    i$giveKSlowestTasks = 1;
    While[i$giveKSlowestTasks <= numTasks && count$giveKSlowestTasks < k,
      If[ContainsAny[validTasks, {inputSortedDecreasing[[i$giveKSlowestTasks]]}],
        kSlowest$giveKSlowestTasks = Append[kSlowest$giveKSlowestTasks, inputSortedDecreasing[[i$giveKSlowestTasks]]];
        count$giveKSlowestTasks++;
      ];
      i$giveKSlowestTasks++;
    ];
    kSlowest$giveKSlowestTasks
  ]
)
```
```Mathematica
decreasingTimeAlgorithm[] := (
  remainingTasks = tasks;
  currentlyValidTasks = {};
  depMatCopy = dependencyMatrix;

  workerAssignments = Table[{}, {i, numTeams}];
  workerEndTimes = Table[0, {i, numTeams}];

  time = 0;
  While[! ContainsOnly[remainingTasks, {}] || ! ContainsOnly[currentlyValidTasks, {}],
    For[i = 1, i <= numTeams, i++,
      If[workerEndTimes[[i]] == time,
        prevTaskIndex = index[Last[workerAssignments[[i]], 0]];
        If[prevTaskIndex != -1,
          For[j = 1, j <= numTasks, j++, depMatCopy[[j, prevTaskIndex]] = False]
        ]
      ]
    ];

    validTasksReturnValue = addValidTasksFromList[remainingTasks, depMatCopy, currentlyValidTasks];
    remainingTasks = validTasksReturnValue[[1]];
    currentlyValidTasks = validTasksReturnValue[[2]];

    workerCountAvailable = 0;
    For[i = 1, i <= numTeams, i++, If[workerEndTimes[[i]] <= time, workerCountAvailable++]];
    slowestTasks = giveKSlowestTasks[currentlyValidTasks, workerCountAvailable];

    numTasksToGive = Dimensions[slowestTasks][[1]];
    For[i = 1, i <= numTeams && numTasksToGive > 0, i++,
      If[workerEndTimes[[i]] <= time,
        workerAssignments[[i]] = Append[workerAssignments[[i]], slowestTasks[[1]]];
        workerEndTimes[[i]] = time + times[[index[slowestTasks[[1]]]]];
        numTasksToGive--;
        currentlyValidTasks = Delete[currentlyValidTasks, index[slowestTasks[[1]], currentlyValidTasks]];
        slowestTasks = Delete[slowestTasks, 1];
      ]
    ];

    For[, i <= numTeams, i++,
      If[workerEndTimes[[i]] <= time,
        assignmentLen = Dimensions[workerAssignments[[i]]][[1]];
        If[assignmentLen == 0,
          workerAssignments[[i]] = {0};
          assignmentLen = 1;
        ];
        If[index[workerAssignments[[i, assignmentLen]]] != -1,
          workerAssignments[[i]] = Append[workerAssignments[[i]], 1],
          workerAssignments[[i, assignmentLen]]++;
        ]
      ]
    ];
    time++;
  ];
  finalEndTime = Max[workerEndTimes];
  For[i = 1, i <= numTeams, i++,
    If[workerEndTimes[[i]] < finalEndTime,
      assignmentLen = Dimensions[workerAssignments[[i]]][[1]];
      If[assignmentLen == 0,
        workerAssignments[[i]] = {0};
        assignmentLen = 1;
      ];
      If[index[workerAssignments[[i, assignmentLen]]] != -1,
        workerAssignments[[i]] = Append[workerAssignments[[i]], finalEndTime - workerEndTimes[[i]]],
        workerAssignments[[i, assignmentLen]] = finalEndTime - workerEndTimes[[i]];
      ];
      workerEndTimes[[i]] = finalEndTime;
    ]
  ];
  {workerAssignments, finalEndTime}
)
```

## Critical Path Algorithm
