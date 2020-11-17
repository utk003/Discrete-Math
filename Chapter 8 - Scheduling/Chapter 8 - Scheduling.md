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
For[i$dependencyMaker = 1, i$dependencyMaker <= numTasks, i$dependencyMaker++,
  dependenciesLength = Dimensions[dependencies[[i$dependencyMaker]]][[1]];
  For[j$dependencyMaker = 1, j$dependencyMaker <= dependenciesLength, j$dependencyMaker++,
    dependencyMatrix[[i$dependencyMaker, index[dependencies[[i$dependencyMaker, j$dependencyMaker]]]]] = True
  ]
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

giveKSlowestTasks[validTasks_, k_, sortedTasks_] := (
  If[Dimensions[validTasks][[1]] <= k,
    validTasks,
    kSlowest$giveKSlowestTasks = {};
    count$giveKSlowestTasks = 0;
    i$giveKSlowestTasks = 1;
    While[i$giveKSlowestTasks <= numTasks && count$giveKSlowestTasks < k,
      If[ContainsAny[validTasks, {sortedTasks[[i$giveKSlowestTasks]]}],
        kSlowest$giveKSlowestTasks = Append[kSlowest$giveKSlowestTasks, sortedTasks[[i$giveKSlowestTasks]]];
        count$giveKSlowestTasks++;
      ];
      i$giveKSlowestTasks++;
    ];
    kSlowest$giveKSlowestTasks
  ]
)
```
```Mathematica
decreasingTimeAlgorithm[sortedTasks_] := (
  remainingTasks$decreasingTimeAlgorithm = tasks;
  currentlyValidTasks$decreasingTimeAlgorithm = {};
  depMatCopy$decreasingTimeAlgorithm = dependencyMatrix;

  workerAssignments$decreasingTimeAlgorithm = Table[{}, {i, numTeams}];
  workerEndTimes$decreasingTimeAlgorithm = Table[0, {i, numTeams}];

  time$decreasingTimeAlgorithm = 0;
  While[! ContainsOnly[remainingTasks$decreasingTimeAlgorithm, {}] || ! ContainsOnly[currentlyValidTasks$decreasingTimeAlgorithm, {}],
    For[i$decreasingTimeAlgorithm = 1, i$decreasingTimeAlgorithm <= numTeams, i$decreasingTimeAlgorithm++,
      If[workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] == time$decreasingTimeAlgorithm,
        prevTaskIndex$decreasingTimeAlgorithm = index[Last[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]], 0]];
        If[prevTaskIndex$decreasingTimeAlgorithm != -1,
          For[j$decreasingTimeAlgorithm = 1, j$decreasingTimeAlgorithm <= numTasks, j$decreasingTimeAlgorithm++,
            depMatCopy$decreasingTimeAlgorithm[[j$decreasingTimeAlgorithm, prevTaskIndex$decreasingTimeAlgorithm]] = False
          ]
        ]
      ]
    ];

    validTasksReturnValue$decreasingTimeAlgorithm = addValidTasksFromList[remainingTasks$decreasingTimeAlgorithm, depMatCopy$decreasingTimeAlgorithm, currentlyValidTasks$decreasingTimeAlgorithm];
    remainingTasks$decreasingTimeAlgorithm = validTasksReturnValue$decreasingTimeAlgorithm[[1]];
    currentlyValidTasks$decreasingTimeAlgorithm = validTasksReturnValue$decreasingTimeAlgorithm[[2]];

    workerCountAvailable$decreasingTimeAlgorithm = 0;
    For[i$decreasingTimeAlgorithm = 1, i$decreasingTimeAlgorithm <= numTeams, i$decreasingTimeAlgorithm++,
      If[workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] <= time$decreasingTimeAlgorithm,
        workerCountAvailable$decreasingTimeAlgorithm++
      ]
    ];
    slowestTasks$decreasingTimeAlgorithm = giveKSlowestTasks[currentlyValidTasks$decreasingTimeAlgorithm, workerCountAvailable$decreasingTimeAlgorithm, sortedTasks];

    numTasksToGive$decreasingTimeAlgorithm = Dimensions[slowestTasks$decreasingTimeAlgorithm][[1]];
    For[i$decreasingTimeAlgorithm = 1, i$decreasingTimeAlgorithm <= numTeams && numTasksToGive$decreasingTimeAlgorithm > 0, i$decreasingTimeAlgorithm++,
      If[workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] <= time$decreasingTimeAlgorithm,
        workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = Append[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]], slowestTasks$decreasingTimeAlgorithm[[1]]];
        workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = time$decreasingTimeAlgorithm + times[[index[slowestTasks$decreasingTimeAlgorithm[[1]]]]];
        numTasksToGive$decreasingTimeAlgorithm--;
        currentlyValidTasks$decreasingTimeAlgorithm = Delete[currentlyValidTasks$decreasingTimeAlgorithm, index[slowestTasks$decreasingTimeAlgorithm[[1]], currentlyValidTasks$decreasingTimeAlgorithm]];
        slowestTasks$decreasingTimeAlgorithm = Delete[slowestTasks$decreasingTimeAlgorithm, 1];
      ]
    ];

    For[, i$decreasingTimeAlgorithm <= numTeams, i$decreasingTimeAlgorithm++,
      If[workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] <= time$decreasingTimeAlgorithm,
        assignmentLen$decreasingTimeAlgorithm = Dimensions[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]]][[1]];
        If[assignmentLen$decreasingTimeAlgorithm == 0,
          workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = {0};
          assignmentLen$decreasingTimeAlgorithm = 1;
        ];
        If[index[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm, assignmentLen$decreasingTimeAlgorithm]]] != -1,
          workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = Append[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]], 1],
          workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm, assignmentLen$decreasingTimeAlgorithm]]++;
        ]
      ]
    ];
    time$decreasingTimeAlgorithm++;
  ];
  finalEndTime$decreasingTimeAlgorithm = Max[workerEndTimes$decreasingTimeAlgorithm];
  For[i$decreasingTimeAlgorithm = 1, i$decreasingTimeAlgorithm <= numTeams, i$decreasingTimeAlgorithm++,
    If[workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] < finalEndTime$decreasingTimeAlgorithm,
      assignmentLen$decreasingTimeAlgorithm = Dimensions[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]]][[1]];
      If[assignmentLen$decreasingTimeAlgorithm == 0,
        workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = {0};
        assignmentLen$decreasingTimeAlgorithm = 1;
      ];
      If[index[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm, assignmentLen$decreasingTimeAlgorithm]]] != -1,
        workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = Append[workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]], finalEndTime$decreasingTimeAlgorithm - workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]]],
        workerAssignments$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm, assignmentLen$decreasingTimeAlgorithm]] = finalEndTime$decreasingTimeAlgorithm - workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]];
      ];
      workerEndTimes$decreasingTimeAlgorithm[[i$decreasingTimeAlgorithm]] = finalEndTime$decreasingTimeAlgorithm;
    ]
  ];
  {workerAssignments$decreasingTimeAlgorithm, finalEndTime$decreasingTimeAlgorithm}
)
```

## Critical Path Algorithm
### Helper Methods
```Mathematica
addNonDependents[depMat_, addToList_] := (
  addToList$addNonDependents = addToList;
  For[i$addNonDependents = 1, i$addNonDependents <= numTasks, i$addNonDependents++,
    If[ContainsAny[addToList$addNonDependents, {tasks[[i$addNonDependents]]}], Continue[]];
    If[ContainsAny[Table[depMat[[k, i$addNonDependents]], {k, numTasks}], {True}], Continue[]];
    addToList$addNonDependents = Append[addToList$addNonDependents, tasks[[i$addNonDependents]]];
  ];
  addToList$addNonDependents
)
```

### Backflow Algorithm
```Mathematica
getCriticalTimes[] := (
  depMatCopy$getCriticalTimes = dependencyMatrix;
  criticalTimes$getCriticalTimes = Table[{}, {k, numTasks}];
  workingOptions$getCriticalTimes = addNonDependents[depMatCopy$getCriticalTimes, {}];
  For[i$getCriticalTimes = 1, i$getCriticalTimes <= numTasks, i$getCriticalTimes++,
    max$getCriticalTimes = 0;
    ind$getCriticalTimes = index[workingOptions$getCriticalTimes[[i$getCriticalTimes]]];
    For[j$getCriticalTimes = 1, j$getCriticalTimes <= numTasks, j$getCriticalTimes++,
      If[dependencyMatrix[[j$getCriticalTimes, ind$getCriticalTimes]],
        max$getCriticalTimes = Max[max$getCriticalTimes, criticalTimes$getCriticalTimes[[j$getCriticalTimes, 2]]]
      ];
    ];
    criticalTimes$getCriticalTimes[[ind$getCriticalTimes]] = {tasks[[ind$getCriticalTimes]], max$getCriticalTimes + times[[ind$getCriticalTimes]]};
    For[j$getCriticalTimes = 1, j$getCriticalTimes <= numTasks, j$getCriticalTimes++,
      depMatCopy$getCriticalTimes[[ind$getCriticalTimes, j$getCriticalTimes]] = False;
    ];
    workingOptions$getCriticalTimes = addNonDependents[depMatCopy$getCriticalTimes, workingOptions$getCriticalTimes];
  ];
  criticalTimes$getCriticalTimes
)
```

## Output
### Helper Method
```Mathematica
interpretAssignments[assignments_, totTime_] := (
  interp$interpretAssignments = Table[0, {k, totTime}];
  ind$interpretAssignments = 0;
  start$interpretAssignments = 0;
  currLength$interpretAssignments = 0;
  currText$interpretAssignments = "";
  For[t$interpretAssignments = 1, t$interpretAssignments <= totTime, t$interpretAssignments++,
    If[t$interpretAssignments > start$interpretAssignments + currLength$interpretAssignments,
      ind$interpretAssignments++;
      start$interpretAssignments = t$interpretAssignments - 1;
      index$interpretAssignments = index[assignments[[ind$interpretAssignments]]];
      If[index$interpretAssignments != -1,
        currLength$interpretAssignments = times[[index$interpretAssignments]];
        currText$interpretAssignments = assignments[[ind$interpretAssignments]],
        currLength$interpretAssignments = assignments[[ind$interpretAssignments]];
        currText$interpretAssignments = "Idle";
      ];
    ];
    interp$interpretAssignments[[t$interpretAssignments]] = currText$interpretAssignments;
  ];
  interp$interpretAssignments
)

printOutput[result_] := (
  Print["Time taken: ", result[[2]]];
  Print["Task assignments:"];
  Print[Prepend[
      Table[Prepend[interpretAssignments[result[[1, k]], result[[2]]], "Team " <> ToString[k]], {k, numTeams}],
      Prepend[Table[k, {k, result[[2]]}], "Team #"]
    ] // TableForm
  ];
)
```

### Decreasing Time Algorithm Output

```Mathematica
inputSortedTrueTime = Sort[input, #1[[2]] > #2[[2]] &];
inputSortedTrueTime = Table[inputSortedTrueTime[[i, 1]], {i, numTasks}];
printOutput[decreasingTimeAlgorithm[inputSortedTrueTime]]
```

### Critical Path Algorithm Output
```Mathematica
inputSortedCriticalTime = Sort[getCriticalTimes[], #1[[2]] > #2[[2]] &];
inputSortedCriticalTime = Table[inputSortedCriticalTime[[i, 1]], {i, numTasks}];
printOutput[decreasingTimeAlgorithm[inputSortedCriticalTime]]
```
