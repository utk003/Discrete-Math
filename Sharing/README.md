# Sharing
## Method of Sealed Bids
### Input Bids Here
```Mathematica
input = {
  {"", "Ferial", "Billy", "Angel"},
  {"Ocean Fisker", 75000, 50000, 35000},
  {"Home", 150000, 170000, 150000},
  {"Vacation Home", 500000, 750000, 800000},
  {"2025 VW Van", 50000, 55000, 30000}
};
input // TableForm
```

### Calculate Metadata and Trim Data Table (Do NOT Modify)
```Mathematica
numPeople = Dimensions[input][[2]] - 1;
numItems = Dimensions[input][[1]] - 1;

people = Drop[input[[1]], 1];
items = Table[input[[i]][[1]], {i, 2, numItems + 1}];

ratings = Table[Table[input[[i]][[j]], {j, 2, numPeople + 1}], {i, 2, numItems + 1}];
```

### Calculate Fair Shares and Item Distributions (Do NOT Modify)
```Mathematica
fairShares = Total[ratings]/numPeople;
itemOwners = Table[FirstPosition[ratings[[i]], Max[ratings[[i]]]][[1]], {i, 1, numItems}];
itemDistribution = Table[Table[If[j == itemOwners[[i]], 1, 0], {j, 1, numPeople}], {i, 1, numItems}];

itemsReceived = Table["", numPeople];
For[i = 1, i <= numItems, i++, itemsReceived[[itemOwners[[i]]]] = itemsReceived[[itemOwners[[i]]]] <> items[[i]] <> ", "]
```

### Calculate Money in Pot/Surplus (Do NOT Modify)
```Mathematica
valueReceived = Total[itemDistribution*ratings];
paidToPot = valueReceived - fairShares // N;
leftInPot = Total[paidToPot];
paidToPot -= leftInPot/numPeople;
```

### Output Final Item and Money Distributions (Do NOT Modify)
```Mathematica
translateToText[money_] := If[money > 0, " paid $" <> ToString[money] <> " to the pot.", " received $" <> ToString[-money] <> " from the pot."]
For[i = 1, i <= numPeople, i++, Print[people[[i]] <> " received " <> itemsReceived[[i]] <> "and" <> translateToText[paidToPot[[i]]]]]
```
