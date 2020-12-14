# Fractals
## Polygon Setup
```Mathematica
getPolygon[numSides_, scale_] := Module[{allcorners},
  allcorners = {};
  For[i = 0, i < numSides, i++,
    allcorners = Append[allcorners,
      scale {
        Cos[((2 i + 1)/numSides - 1/2) \[Pi]],
        Sin[((2 i + 1)/numSides - 1/2) \[Pi]]
      } // N
    ]
  ];
  allcorners
]
equiTriangle = getPolygon[3, 10];
square = getPolygon[4, 10];
```
## Koch Snowflake and Anti-Snowflake (Pure Generation)
```Mathematica
getRotMatScaler[angleRads_] := RotationMatrix[angleRads]/(2 Cos[angleRads])
```
```Mathematica
generateCurvePaths[sides_, depth_, angle_] := Module[{res, len, i},
  rotMat = getRotMatScaler[angle];
  rat = 1/2 1/(Cos[angle] + 1);
  res = {};
  If[depth <= 0,
    len = Dimensions[sides][[1]];
    For[i = 1, i < len, i++,
      res = Append[res, {sides[[i]], sides[[i + 1]]}]
    ],
    len = Dimensions[sides][[1]];
    For[i = 1, i < len, i++,
      p1 = (1 - rat) sides[[i]] + rat sides[[i + 1]];
      p2 = rat sides[[i]] + (1 - rat) sides[[i + 1]];
      p3 = p1 + rotMat.(p2 - p1);
      res = Join[res,
        generateCurvePaths[
          {sides[[i]], p1, p3, p2, sides[[i + 1]]},
          depth - 1, angle
        ]
      ];
    ];
  ];
  res
]
```
```Mathematica
generateSnowflakeOverlays[sides_, depth_, angle_] := Module[{res, len, i},
  rotMat = getRotMatScaler[angle];
  res = {};
  If[depth > 0,
    len = Dimensions[sides][[1]];
    For[i = 1, i < len, i++,
      p1 = (2 sides[[i]] + sides[[i + 1]])/3;
      p2 = (sides[[i]] + 2 sides[[i + 1]])/3;
      p3 = p1 + rotMat.(p2 - p1);
      res = Append[res, {p1, p2, p3}];
      res = Join[res,
        generateSnowflakeOverlays[
          {sides[[i]], p1, p3, p2, sides[[i + 1]]},
          depth - 1, angle
        ]
      ];
    ];
  ];
  res
]
```
```Mathematica
Graphics[{Black, Line[generateCurvePaths[{{-5, 0}, {5, 0}}, 5, \[Pi]/3]]}]
```
![Koch Curve](graphics/Koch%20Curve.png)
```Mathematica
Graphics[{
  Black, Triangle[equiTriangle],
  Black, Triangle[generateSnowflakeOverlays[
    Append[equiTriangle, equiTriangle[[1]]], 5, -(\[Pi]/3)
  ]]
}]
```
![Koch Snowflake](graphics/Koch%20Snowflake.png)
```Mathematica
Graphics[{
  Black, Triangle[equiTriangle],
  White, Triangle[generateSnowflakeOverlays[
    Append[equiTriangle, equiTriangle[[1]]], 5, (\[Pi]/3)
  ]]
}]
```
![Koch Anti-Snowflake](graphics/Koch%20Anti-Snowflake.png)

## Cesàro Curve, Snowflake, and Anti-Snowflake (Pure Generation)
```Mathematica
cesaroAngle = 60;
```
```Mathematica
Graphics[{
  Black, Triangle[square],
  Black, Triangle[generateSnowflakeOverlays[
    Append[square, square[[1]]], 6, -cesaroAngle \[Pi]/180
  ]]
}]
```
![Cesaro Snowflake](graphics/Cesaro%20Snowflake.png)
```Mathematica
Graphics[{
  Black, Triangle[square],
  White, Triangle[generateSnowflakeOverlays[
    Append[square, square[[1]]], 6, cesaroAngle \[Pi]/180
  ]]
}]
```
![Cesaro Anti-Snowflake](graphics/Cesaro%20Anti-Snowflake.png)

## Sierpiński Triangle (Pure Generation)
```Mathematica
generateOverlays[triangle_, depth_] := Module[{res, mid},
  res = {};
  If[depth > 0,
    mid = {triangle[[1]] + triangle[[2]], triangle[[2]] + triangle[[3]], triangle[[3]] + triangle[[1]]}/2;
    res = Append[res, mid];
    res = Join[res, generateOverlays[{triangle[[1]], mid[[1]], mid[[3]]}, depth - 1]];
    res = Join[res, generateOverlays[{triangle[[2]], mid[[1]], mid[[2]]}, depth - 1]];
    res = Join[res, generateOverlays[{triangle[[3]], mid[[2]], mid[[3]]}, depth - 1]];
  ];
  res
]
```
```Mathematica
Graphics[{
  Black, Triangle[equiTriangle],
  White, Triangle[generateOverlays[equiTriangle, 6]]
}]
```
![Sierpinski Triangle](graphics/Sierpinski%20Triangle.png)

## Sierpiński Triangle (Twisted Generation)
```Mathematica
generateTwistedOverlays[triangle_, depth_] := Module[{res, mid},
  sides = Append[triangle, triangle[[1]]];
  res = {};
  If[depth <= 0, res = {triangle},
    len = Dimensions[sides][[1]];
    mid = {};
    For[i = 1, i < len, i++,
      size = sides[[i + 1]] - sides[[i]];
      size = Sqrt[size[[1]]^2 + size[[2]]^2];
      mid = Append[mid,
        (sides[[i]] + sides[[i + 1]])/2 + {
          RandomReal[amplitude size {-1, 1}],
          RandomReal[amplitude size {-1, 1}]
        }
      ];
    ];
    res = Join[res, generateTwistedOverlays[{triangle[[1]], mid[[1]], mid[[3]]}, depth - 1]];
    res = Join[res, generateTwistedOverlays[{triangle[[2]], mid[[1]], mid[[2]]}, depth - 1]];
    res = Join[res, generateTwistedOverlays[{triangle[[3]], mid[[2]], mid[[3]]}, depth - 1]];
  ];
  res
]
```
```Mathematica
amplitude = 0.1;
Graphics[{
  White, Triangle[equiTriangle],
  Black, Triangle[generateTwistedOverlays[equiTriangle, 6]]
}]
```
![Twisted Sierpinski Triangle](graphics/Twisted%20Sierpinski%20Triangle.png)

## Sierpiński Carpet (Pure Generation)
```Mathematica
generateCarpetOverlays[square_, depth_] := Module[{res, mid, d1, d2, c, i, j},
  res = {};
  If[depth > 0,
    d1 = (square[[2]] - square[[1]])/3;
    d2 = (square[[4]] - square[[1]])/3;
    For[i = 0, i < 3, i++,
      For[j = 0, j < 3, j++,
        If[i != 1 || j != 1,
          c = square[[1]] + i d1 + j d2;
          res = Join[res, generateCarpetOverlays[{c, c + d1, c + d1 + d2, c + d2}, depth - 1]];
        ];
      ];
    ];
    c = square[[1]] + d1 + d2;
    res = Append[res, {c, c + d1, c + d1 + d2, c + d2}];
  ];
  res
]
```
```Mathematica
Graphics[{
  Black, Polygon[square],
  White, Polygon[generateCarpetOverlays[square, 5]]
}]
```
![Sierpinski Carpet](graphics/Sierpinski%20Carpet.png)

## Sierpiński Triangle (Chaos Game Generation)
```Mathematica
startPoint = {0, 0};
allpoints = {};
For[i = 1, i <= 10000, i++,
  allpoints = Append[allpoints, startPoint];
  startPoint = (startPoint + RandomChoice[equiTriangle])/2;
]
ListPlot[allpoints]
```
![Sierpinski Triangle (Chaos)](graphics/Sierpinski%20Triangle%202.png)

## Sierpiński Triangle Square Variant (Chaos Game Generation)
```Mathematica
startPoint = {0, 0};
allpoints = {};
rat = RandomReal[]
For[i = 1, i <= 10000, i++,
  allpoints = Append[allpoints, startPoint];
  startPoint = (1 - rat) startPoint + rat RandomChoice[square];
]
ListPlot[allpoints]
```

## Mandelbrot Sequence
```Mathematica
seed = (1 + I)/3;
seq = {seed};
curr = seed // N;
For[i = 1, i <= 100, i++, curr = curr^2 + seed; seq = Append[seq, curr]]
coords = Table[{Re[seq[[k]]], Im[seq[[k]]]}, {k, Dimensions[seq][[1]]}];
ComplexListPlot[seq]
ListLinePlot[coords]
```
![Mandelbrot Sequence](graphics/Mandelbrot%20Sequence.png)
![Mandelbrot Sequence (Connected)](graphics/Mandelbrot%20Sequence%202.png)

## Mandelbrot Set
```Mathematica
mand[curr_, seed_] := mand[curr, seed] = curr^2 + seed
mand2[ind_, seed_] := mand2[ind, seed] =
  If[ind > 0,
    If[Abs[mand2[ind - 1, seed]] > 10^9,
      \[Infinity],
      mand2[ind - 1, seed]^2 + seed
    ],
    seed
  ]
```
```Mathematica
check[a_, b_] := check[a, b] = (
  seed = a + b I;
  If[mand2[101, seed] != \[Infinity], Return[Black]];
  For[i = 1, i <= 25, i++,
    If[mand2[i, seed] == \[Infinity], Break[]]
  ];
  Return[RGBColor[i/26, 0, 1 - i/26, 2/3]];
)
```
```Mathematica
set = {};
colorList = {};
resolution = 0.03;
For[a = -2, a <= 1, a += resolution,
  For[b = -1.5, b <= 1.5, b += resolution,
    set = Append[set, {{a, b}}];
    colorList = Append[colorList, check[a, b]];
  ]
]
```
```Mathematica
ListPlot[set, PlotStyle -> colorList, AxesOrigin -> {-2.1, -1.6}]
```
![Mandelbrot Set](graphics/Mandelbrot%20Set.png)
