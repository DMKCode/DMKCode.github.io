---
title: Using Jest snapshots to simply redux tests
date: 2017-08-12 10:23:20
tags: ['redux', 'jest', 'react']
---

When writing tests for redux actions and reducers, its often the case when refactoring actions and reducers, tests are broken and manual changes need to be made to the tests to reflect these changes. In this post I'm going to describe a simple way to write concise tests which allow for easy refactoring using [Jest](https://facebook.github.io/jest/) snapshots. 

## Case Point
Lets assume, you have an action creator defined in some file called 'actions.js'.
This action creator expects a numeric data type which is then parsed to floating point number and attached to the action - something along these lines... 

#### actions.js
```javascript
export const CONVERT_NUMBER = 'CONVERT_NUMBER';

export const convertNumber = (num) => ({
  type: CONVERT_NUMBER,
  floatingPointNumber: parseFloat(num),
});

```

In the test for this action creator, you would have something along the lines of...

#### actions.test.js
```javascript
import { CONVERT_NUMBER, convertNumber } from '../actions/actions.js';

it('creates a CONVERT_NUMBER action', () => {
  const expected = {
    type: CONVERT_NUMBER,
    floatingPointNumber: parseFloat(100),
  };
  const actual = convertNumber();
  expect(actual).toEqual(expected);
});

```


This form of testing works and perfectly suitable for most situations. However, once we make any changes to the action creator, we would need to manually update the test to reflect the same changes.
What Jest provides is a simple way to automate the updates to the tests using snapshots.

To make use of snapshots we would change the test above like so...

#### actions.test.js
```javascript
it('creates a CONVERT_NUMBER action', () => {
  expect(convertNumber()).toMatchSnapshot();
});

```

On first run of the test, Jest will create a snapshot in the '__snapshots' directory relative to the test file. When updates are made to the action creator such as removing 'parseFloat', the test will fail with a description that the action creator has changed, giving you the opportunity to update the snapshot if the changes are deemed to be appropriate / intended.

I hope you can see how snapshots can help in writing concise tests for your applications.

Thanks for reading!

