# Squarify
This package is a TypeScript implementation (with no external runtime dependencies) of [Bruls _et al._'s squarified tree map algorithm](https://graphics.ethz.ch/teaching/scivis_common/Literature/squarifiedTreeMaps.pdf).
If you're interested in how the algorithm works, I explain it in details in an [article on my blog](https://www.huy.dev/squarified-tree-map-reasonml-part-1-2019-03/).

This is a "battle-tested" implementation and is currently used to calculate the layout of the trade tree map in the [Atlas of Economic Complexity](http://atlas.cid.harvard.edu/explore/), a data visualizsation tool used by 15,000 unique visitors per month.

Unlike other JavaScript implementations, it is written in clear, readable code and backed up by unit tests ([98% coverage](https://codecov.io/gh/huy-nguyen/squarify)).

As a strong believer in composable software, I deliberately made this package minimal. It only performs the layout step. You are free to use the output to render whichever way you want.


[![npm](https://img.shields.io/npm/v/squarify.svg?style=flat-square)](https://www.npmjs.com/package/squarify)
[![tested with jest](https://img.shields.io/badge/tested_with-jest-99424f.svg)](https://github.com/facebook/jest)
[![codecov](https://codecov.io/gh/huy-nguyen/squarify/branch/master/graph/badge.svg)](https://codecov.io/gh/huy-nguyen/squarify) [![Greenkeeper badge](https://badges.greenkeeper.io/huy-nguyen/squarify.svg)](https://greenkeeper.io/)

## Installation

`npm install --save squarify`

## Usage

### Input
The default `export` of this package is a function that expects two parameters:
- An array of input `data`. It's a recursive data structure where each element has this shape:
```typescript
type Input<Custom> = {
    value: number;
    children?: Input<Custom>[];
} & Custom;
```
where `Custom` describes the type of any extra data the user wants to attach to each node. This data will be passed through to the result.
  - `value` is a **strictly positive** (i.e. non-zero) number and must be provided. The displayed area of any node is proportional to its `value`. The sum of the `value` of a node's leaves must equal the `value` of the node itself. At every level of nesting of `data`, all array items must be already sorted in descending `value` order.
  - `children` is optional and indicates whether a datum is a node (`children` is an array) or a leaf (`children` is `undefined`).
  - Your data also shouldn't contain the property `normalizedValue` because it is used internally by the package.

Sample input data (note that the `name` and `color` fields in the input data, which are user-defined and optional, will be passed through to the result):
```javascript
[{
  name: 'Azura', value: 6, color: 'red',
}, {
  name: 'Seth', value: 5, color: '',
  children: [
    {
      name: 'Noam', value: 3, color: 'orange',
    },
    {
      name: 'Enos', value: 2, color: 'yellow',
    },
  ]
}, {
  name: 'Awan', value: 5, color: '',
  children: [{
      name: 'Enoch', value: 5, color: 'green',
  }]
}, {
  name: 'Abel', value: 4, color: 'blue',
}, {
  name: 'Cain', value: 1, color: 'indigo',
}]
```
- A rectangle that this algorithm will try to fit the tree map into. It should be specified as an object with this shape:
```typescript
interface Container {
    x0: number;
    y0: number;
    x1: number;
    y1: number;
}
```
where (`x0`, `y0`) and (`x1`, `y1`) are the coordinates of the top-left and bottom-right corners of the rectangle, respectively (`x` increases going rightward and `y` increases going downward on the page). Sample data:
```js
{x0: 0, y0: 0, x1: 100, y1: 50};
```

### Output
The output is an array of layout rectangles. Each rectangle has this shape:
```typescript
interface Result {
  x0: number;
  y0: number;
  x1: number;
  y1: number;
  value: number,
  normalizedValue: number
} & Custom
```
where
  - `x0`, `y0`, `x1`, `y1` are the coordinates of the top-left and bottom-right corners of the rectangle.
  - `normalizedValue` is a value used internally, which you can ignore.
  - `value` is the same one from the original input data.
  - Any extra properties in the input are passed through to this rectangle. Also note that the algorithm also flatten the output such that only leaves in the original data will appear in the output.

Sample output for the above sample input:
```javascript
[
  {x0: 0, y0: 0, x1: 41.66, y1: 35, name: 'Noam', value: 3, color: 'orange'},
  {x0: 0, y0: 35, x1: 41.66, y1: 50, name: 'Enos', value: 2, color: 'yellow'},
  {x0: 41.66, y0: 0, x1: 70.83, y1: 50, name: 'Abel', value: 4, color: 'blue'},
  {x0: 70.83, y0: 0, x1: 100, y1: 28.57, name: 'Azura', value: 6, color: 'red'},
  {x0: 70.83, y0: 0, x1: 90.27, y1: 50, name: 'Enoch', value: 5, color: 'green'},
  {x0: 90.27, y0: 28.57, x1: 100, y1: 50, name: 'Cain', value: 1, color: 'indigo'}
]
```

### Sample input
This is sample usage in a TypeScript file:
```typescript
import squarify, {
  Input
} from 'squarify'

interface Custom {
  name: string;
  color: string;
}
const data: Input<Custom>[] = [{
  name: 'Azura', value: 6, color: 'red',
}, {
  name: 'Seth', value: 5, color: '',
  children: [
    {
      name: 'Noam', value: 3, color: 'orange',
    },
    {
      name: 'Enos', value: 2, color: 'yellow',
    },
  ]
}, {
  name: 'Awan', value: 5, color: '',
  children: [{
      name: 'Enoch', value: 5, color: 'green',
  }]
}, {
  name: 'Abel', value: 4, color: 'blue',
}, {
  name: 'Cain', value: 1, color: 'indigo',
}];

const container = {x0: 0, y0: 0, x1: 100, y1: 50};

const output = squarify<Custom>(data, container);
```

This is a sample in JavaScript:
```javascript
import squarify from 'squarify'
// Or `const squarify = require('squarify')` in NodeJS.

const data = [{
  name: 'Azura', value: 6, color: 'red',
}, {
  name: 'Seth', value: 5, color: '',
  children: [
    {
      name: 'Noam', value: 3, color: 'orange',
    },
    {
      name: 'Enos', value: 2, color: 'yellow',
    },
  ]
}, {
  name: 'Awan', value: 5, color: '',
  children: [{
      name: 'Enoch', value: 5, color: 'green',
  }]
}, {
  name: 'Abel', value: 4, color: 'blue',
}, {
  name: 'Cain', value: 1, color: 'indigo',
}];

const container = {x0: 0, y0: 0, x1: 100, y1: 50};

const output = squarify(data, container);
```

## Contributing
Please see the [contributing guide](CONTRIBUTING.md) if you are interested in helping.
