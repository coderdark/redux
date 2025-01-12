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
+ Reducers - Functions that receive the current state and an action as inputs and produce a new state as the output.
  - `state`
  - `action`, the only requirement for an action is to have a `type`.  You can append other properties like `payload`
```
//Example 1
const reducer = (state, action) => {
  return state;
}

//Example 2
const reducer = (state = initialState, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return {...state, value: state.value + 1};
        case 'DECREMENT':
            return {...state, value: state.value - 1};
    }

    return state;
};

//Example 3
const reducer = (state = initialState, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return {...state, value: state.value + action.payload};
        case 'DECREMENT':
            return {...state, value: state.value + action.payload};
    }

    return state;
};
```
+ Action Creators - A simple function where you pass the payload as an argument and return an object with the state and the action as properties.  This also helps when refactoring your code.  If you did not use an action creator but just declare an action like this `const incrementAction = {type: 'INCREMENT'};` all over your code, when a change is needed you have to go and search your code to change the action.  Best practice is to use an action creator.
```
const increase = (amount) => ({type: 'INCREMENT', payload: amount});
```
+ Dispatch - this function executes the action and dispaches its return to trigger a state change.
```
store.dispatch(increase());
```
## Full Example
```
import {createStore, compose, applyMiddleware, bindActionCreators, combineReducers} from 'redux';

const initialState = {value: 0};

const increase = (amount) => ({type: 'INCREMENT'});
const decrease = (amount) => ({type: 'DECREMENT'});

const reducer = (state = initialState, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return {value: state.value + 1};
        case 'DECREMENT':
            return {value: state.value - 1};
    }

    return state;
};

const store = createStore(reducer, initialState);

store.dispatch(increase());
store.dispatch(increase());
store.dispatch(increase());
store.dispatch(increase());
store.dispatch(increase());
store.dispatch(decrease());

console.log(store.getState());
```
