# Redux
Global state management (internally uses react context). Small api set: `applyMiddleware`, `bindActionCreators`, `combineReducers`, `compose`, `createStore` + 4 more.

## API
+ Store - Its the place that keeps your state. Only one store, having multiple store is an unti-pattern. You create one using the `createStore` function.
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
+ Reducers - Functions that receive the current state and an action as inputs and produce a new state as the output. Keep your state as flat as you can or break it down into multiple reducers. Including API responses that you do not control, break the response at the client to be stored in redux. 
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
            return {value: state.value + 1};
        case 'DECREMENT':
            return {value: state.value - 1};
    }

    return state;
};

//Example 3
const reducer = (state = initialState, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return {value: state.value + action.payload};
        case 'DECREMENT':
            return {value: state.value + action.payload};
    }

    return state;
};
```
+ Action Creators - A simple function where you pass the payload as an argument and return an object with the state and the action as properties.  This also helps when refactoring your code.  If you did not use an action creator but just declare an action like this `const incrementAction = {type: 'INCREMENT'};` all over your code, when a change is needed you have to go and search your code to change the action.  Best practice is to use an action creator.
```
const increase = (amount) => ({type: 'INCREMENT', payload: amount});
```
+ `dispatch` - this function executes the action and dispaches its return to trigger a state change.  The dispatch goes thru ALL of the reducers found in the store.
```
store.dispatch(increase());
```
+ `getState` - It returns the state of the store.
```
store.getState();
```
+ `subscribe` - It registers a listener for the store. You pass a function as an argument to be executed any time the state changes. It listens to any changeges of the state.
+ `bindActionCreators` - It registers your functions to be called without having to use `dispatch`.  It basically binds your functions to the object you pass as an argument. This way you can call your functions without the `dispatch` method.  The first object argument with your functions inside as properties, second the dispatch method.
```
const actions = bindActionCreators({increase, decrease}, store.dispatch);

actions.increase();
actions.increase();
actions.increase();
actions.increase();
actions.increase();
actions.decrease();
```
+ `combineReducers` - it combines all your reducers into one, you can then pass the return reducer from this function in the `createStore` function.
```
const initialState = {players: [], teams: []};

const addPlayer = (player) => ({type: 'ADD_PLAYER', payload: player});
const addTeam = (team) => ({type: 'ADD_TEAM', payload: team});

const playersReducer = (players = initialState.players, action) => {
    if (action.type === 'ADD_PLAYER') {
        return [...players, action.payload];
    }

    return players;
};

const teamsReducer = (teams = initialState.teams, action) => {
    if (action.type === 'ADD_TEAM') {
        return [...teams, action.payload];
    }

    return teams;
};

const appReducer = combineReducers({
    players: playersReducer,
    teams: teamsReducer
});

const store = createStore(appReducer, initialState);
```
+ Enhancers - Add additional fuctionality to the redux store. They can be created by you or from a library.  Use `compose` to implement many enhancers.
```
//Custom enhancer
const monitorReducerEnhancer = (createStore) => (reducer, initialState, enhancer) => {
  const monitoredReducer = (state, action) => {
    const start = performance.now();
    const newState = reducer(state, action);
    const end = performance.now();
    const diff = round(end - start);

    console.log("Reducer process time:", diff);

    return newState;
  };

  return createStore(monitoredReducer, initialState, enhancer);
};

const store = createStore(appReducer, stateSnapShotEnhancer);

//If using many enhancers use `componse`
//const store = createStore(appReducer, compose(stateSnapShotEnhancer, myOtherEnhancer));
```
+ Middleware - Prefer way, better than enhancers. Add additional fuctionality to the redux store. They can be created by you or from a library. Use Redux's `applyMiddleware` to implement 1 or many (comma separated).
```
const stateSnapShot = (store) => (next) => (actions) => {
    console.log('Before', store.getState());
    next(actions);
    console.log('After', store.getState());
};

const store = createStore(appReducer, applyMiddleware(stateSnapShot));

//If you have more than one middleware, you can include them all in the applyMiddleware separated by comma.
//const store = createStore(appReducer, applyMiddleware(stateSnapShot));
```

## Full Example (Vanilla JS)
```
import {createStore, compose, applyMiddleware, bindActionCreators, combineReducers} from 'redux';

const initialState = {players: [], teams: []};

const addPlayer = (player) => ({type: 'ADD_PLAYER', payload: player});
const addTeam = (team) => ({type: 'ADD_TEAM', payload: team});

const playersReducer = (players = initialState.players, action) => {
    if (action.type === 'ADD_PLAYER') {
        return [...players, action.payload];
    }

    return players;
};

const teamsReducer = (teams = initialState.teams, action) => {
    if (action.type === 'ADD_TEAM') {
        return [...teams, action.payload];
    }

    return teams;
};

const appReducer = combineReducers({
    players: playersReducer,
    teams: teamsReducer
});

//// This code is using enhancers, better way is to use middleware, see example below.
// const stateSnapShotEnhancer = (createStore) => (reducer, initialState, enhancer) => {
//     const snapShotReducer = (state, action) => {
//         console.log('Before', state);
//
//         const newState = reducer(state, action);
//
//         console.log('After', newState);
//
//         return newState;
//     };
//
//     return createStore(snapShotReducer, initialState, enhancer);
// };

// const store = createStore(appReducer, stateSnapShotEnhancer);

//Middleware are simpler than enhancers
const stateSnapShot = (store) => (next) => (actions) => {
    console.log('Before', store.getState());
    next(actions);
    console.log('After', store.getState());
};

const store = createStore(appReducer, applyMiddleware(stateSnapShot));

store.subscribe(() => console.log(store.getState()));

const actions = bindActionCreators({addPlayer, addTeam}, store.dispatch);

//// Call dispatch if not using the bindActionCreator function
// store.dispatch(addPlayer({name: 'Josh Allen'}));
// store.dispatch(addPlayer({name: 'Joe Montana'}));
// store.dispatch(addTeam({name: 'Buffalo Bills'}));
// store.dispatch(addTeam({name: '49ers'}));

actions.addPlayer({name: 'Josh Allen'});
actions.addTeam({team: 'Buffalo Bills'});

```
