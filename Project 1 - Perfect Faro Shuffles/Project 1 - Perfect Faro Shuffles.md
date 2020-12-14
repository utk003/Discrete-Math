# Shuffling/Perfect (Faro) Shuffles
## Shuffle Functions (Do NOT Modify)
```Mathematica
outshuffle[deck_] := (
  numCards = Dimensions[deck][[1]];
  If[numCards == 1, Return[deck]];
  tempCount = Floor[numCards/2];
  newDeck = deck;
  If[numCards != 2 tempCount, newDeck = Append[newDeck, 0]; tempCount++];
  partition = Partition[newDeck, tempCount];
  first = partition[[1]];
  second = partition[[2]];
  newDeck = {};
  For[i = 1, i <= tempCount, i++,
    newDeck = Append[Append[newDeck, first[[i]]], second[[i]]]
  ];
  If[numCards != 2 tempCount, newDeck = Delete[newDeck, numCards + 1]];
  newDeck
)
inshuffle[deck_] := (
  numCards = Dimensions[deck][[1]];
  If[numCards == 1, Return[deck]];
  tempCount = Floor[numCards/2];
  partition = Partition[deck, tempCount];
  first = partition[[1]];
  second = partition[[2]];
  newDeck = {};
  For[i = 1, i <= tempCount, i++,
    newDeck = Append[Append[newDeck, second[[i]]], first[[i]]]
  ];
  If[numCards != 2 tempCount, newDeck = Append[newDeck, deck[[numCards]]]];
  newDeck
)
shuffle[deck_] := inshuffle[deck]
countShuffles[deck_] := (
  count = 1;
  newDeck = shuffle[deck];
  While[deck != newDeck, newDeck = shuffle[newDeck]; count++];
  count
)
```

## Count Shuffles
### Data Input (Search Bound)
```Mathematica
upperBound = 20;
```

### Search and Catalog (Do NOT Modify)
```Mathematica
shuffle[deck_] := inshuffle[deck];
ins = Table[countShuffles[Table[k, {k, n}]], {n, 1, upperBound}];
shuffle[deck_] := outshuffle[deck];
outs = Table[countShuffles[Table[k, {k, n}]], {n, 1, upperBound}];
shuffleCounts = Table[{n, ins[[n]], outs[[n]]}, {n, 1, upperBound}];
Prepend[shuffleCounts, {"Number of Cards", "Number of Inshuffles", "Number of Outshuffles"}] // TableForm
ListLinePlot[{
  Table[{x, ins[[x]]}, {x, 1, upperBound}],
  Table[{x, outs[[x]]}, {x, 1, upperBound}]
}]
```
![Output](https://github.com/utk003/Discrete-Math/blob/main/Project%201%20-%20Perfect%20Faro%20Shuffles/output.png)
