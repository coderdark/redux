# Redux
Global state management (internally uses react context). Small api set: `applyMiddleware`, `bindActionCreators`, `combineReducers`, `compose`, `createStore` + 4 more.

## API
+ Componse - takes multiple functions and composes a new function that applies all the functions passed from right to left.
```
const fn1 = (string) => string.toUpperCase();
const fn2 = (string) => string.repeat(3);
const fn3 = (string) => string.bold();

const runAllFns = redux.compose(
  fn1,
  fn2,
  fn3
);
```
