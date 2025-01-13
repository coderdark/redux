# Redux
Global state management (internally uses react context). Small api set: `applyMiddleware`, `bindActionCreators`, `combineReducers`, `compose`, `createStore` + 4 more.

## Redux
### API
+ Store - Its the place that keeps your state. Only one store, having multiple store is an unti-pattern. You create one using the `createStore` function.
```
import {applyMiddleware, createStore} from 'redux';
import {counterReducer} from './reducers/index.js';
import {localStorage} from './localStorage.js';

const initialState = {count: 0};

export const store = createStore(counterReducer, initialState);

////If you use middlewares
// export const store = createStore(counterReducer, initialState, applyMiddleware(localStorage));
```
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
+ `dispatch` - this function executes the action and dispaches its return to trigger a state change.  The dispatch goes thru ALL of the reducers found in the store. Use `useDispatch` hook in react.
```
//Vanilla JS
store.dispatch(increase());

//React 
import {useDispatch, useSelector} from 'react-redux';

function App() {
    const dispatch = useDispatch();
    //...
}
```
+ `useSelector` - this hook is a function that returns the full state and then you can select only the property in the state that you need.
```
import './App.css';
import {useSelector} from 'react-redux';

function App() {
    const count = useSelector((state) => state.count);
}

export default App;
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

### Full Example (Vanilla JS)
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

### Full Example (React)
```
//src/redux/actions/index.js
export const increment = () => ({type: 'INCREMENT'});

//src/redux/reducers/index.js
export const counterReducer = (state, action) => {
    if (action.type === 'INCREMENT') {
        return {count: state.count + 1};
    }

    return state;
};

//src/redux/index.js
import {applyMiddleware, createStore} from 'redux';
import {counterReducer} from './reducers/index.js';
import {localStorage} from './localStorage.js';

const initialState = {count: 0};

export const store = createStore(counterReducer, initialState);

//src/redux/provider.js
import {Provider} from 'react-redux';
import {store} from './index.jsx';

export const StoreProvider = ({children}) => {
    return <Provider store={store}>
        {children}
    </Provider>;
};

//src/App.jsx
import './App.css';
import {useDispatch, useSelector} from 'react-redux';
import {increment} from './redux/actions';

function App() {
    const dispatch = useDispatch();
    const count = useSelector((state) => state.count);

    return (
        <button onClick={() => dispatch(increment())}>
            count is {count}
        </button>
    );
}

export default App;
```

## Redux Toolkit - https://redux-toolkit.js.org/
Sits on top of **Redux**, it hides alot of the stuff that redux needs to work. Basically some of the foundation work you did in redux. 
+ Folder (Features) structure is app to your app and what works for you.
+ Includes a helper library called  `nanoid` to create unique ids.
+ Store - use the `configureStore` to set your redux store. When configuring the store, redux-toolkit sets 3 **development** middlewares: Mutation Checking, Serialized State Exporting and Thunk. It also sets developer tools (Chrome Plugin to debug redux) automatically.
```
import { configureStore } from "@reduxjs/toolkit";
import { tasksSlice } from "./TasksSlice";

export const store = configureStore({
  reducer: {
    tasks: tasksSlice.reducer,
  },
});
```
+ Slices - you keep all your redux pieces such as actions creators, reducers, etc in here.
```
import { createSlice, nanoid } from "@reduxjs/toolkit";

const createTask = (title) => ({
  id: nanoid(),
  title,
  complete: false,
  assignedTo: "",
});

const initialState = [
  createTask("Order more energy drinks"),
  createTask("Water the plants"),
];

export const tasksSlice = createSlice({
  name: "tasks",
  initialState,
  reducers: {
    add: (state, action) => {
      const task = createTask(action.payload);
      state.push(task);
    },
  },
});
```
+ Reducers in the slice - You can write "mutating" logic in reducers but it does not actually mutate the state. Redux-Toolkit uses **Immer** a library which catches changes and produces a new immutable state from those changes.
+ `useDispatch` - this function executes the action and dispaches its return to trigger a state change.  The dispatch goes thru ALL of the reducers found in the store.
```
import {useDispatch} from 'react-redux';

function App() {
    const dispatch = useDispatch();
    //...
}
```
+ `useSelector` - this hook is a function that returns the full state and then you can select only the property in the state that you need.
```
import './App.css';
import {useSelector} from 'react-redux';

function App() {
    const count = useSelector((state) => state.count.value);
}

export default App;
```
+`createAction` - use this function to change the way you pass an action to your dispatch.  Instead of using the more verbose way of calling a function `dispatch(taskReducer.actions.toggle)` using `createAction` can help you make the function call to be `dispatch(toggleTask)`.
```
export const toggleTask = createAction("tasks/toggle", (taskId, complete) => ({
  payload: { taskId, complete },
}));
```

## Full Example React Redux-Toolkit
```
//src/redux/slices/counterSlice.js
import {createSlice} from '@reduxjs/toolkit';

export const counterSlice = createSlice({
    name: 'counter',
    initialState: {value: 0},
    reducers: {
        increment: (state, action) => {
            state.value += 1;
        }
    }
});

//src/App.jsx
import './App.css';
import {useDispatch, useSelector} from 'react-redux';
import {counterSlice} from './redux/slices/counterSlice.js';

function App() {
    const dispatch = useDispatch();
    const counter = useSelector((state) => state.counter.value);

    return (
        <button onClick={() => dispatch(counterSlice.actions.increment())}>
            count is {counter}
        </button>
    );
}

export default App;

//src/main.jsx
import {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import App from './App.jsx';
import {Provider} from 'react-redux';
import {store} from './redux/index.js';

createRoot(document.getElementById('root')).render(
    <Provider store={store}>
        <StrictMode>
            <App/>
        </StrictMode>
    </Provider>
);
```
