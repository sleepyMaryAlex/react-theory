# ?configureStore

A friendly abstraction over the standard Redux `createStore` function that adds good defaults to the store setup for a better development experience.

### Parameters

`configureStore` accepts a single configuration object parameter, with the following options:

1. `reducer`

If this is a single function, it will be directly used as the root reducer for the store.

If it is an object of slice reducers, like `{users : usersReducer, posts : postsReducer}`, configureStore will automatically create the root reducer by passing this object to the Redux `combineReducers` utility.

2. `middleware`

An optional array of Redux middleware functions.

If this option is provided, it should contain all the middleware functions you want added to the store. `configureStore` will automatically pass those to `applyMiddleware`.

If not provided, `configureStore` will call `getDefaultMiddleware` and use the array of middleware functions it returns.

Where you wish to add onto or customize the default middleware, you may pass a callback function that will receive `getDefaultMiddleware` as its argument, and should return a middleware array.

3. `devTools`

If this is a `boolean`, it will be used to indicate whether `configureStore` should automatically enable support for the Redux DevTools browser extension.

Defaults to `true`.

The Redux DevTools Extension recently added support for showing action stack traces that show exactly where each action was dispatched. Capturing the traces can add a bit of overhead, so the DevTools Extension allows users to configure whether action stack traces are captured by setting the 'trace' argument. If the DevTools are enabled by passing `true` or an object, then `configureStore` will default to enabling capturing action stack traces in development mode only.

4. `preloadedState`

An optional initial state value to be passed to the Redux `createStore` function.

5. `enhancers`

An optional array of Redux store enhancers, or a callback function to customize the array of enhancers.

If defined as an array, these will be passed to the Redux `compose` function, and the combined enhancer will be passed to `createStore`.

~~~
// file: todos/todosReducer.ts noEmit
import type { Reducer } from "@reduxjs/toolkit";
declare const reducer: Reducer<{}>;
export default reducer;

// file: visibility/visibilityReducer.ts noEmit
import type { Reducer } from "@reduxjs/toolkit";
declare const reducer: Reducer<{}>;
export default reducer;

// file: store.ts
import { configureStore } from "@reduxjs/toolkit";

// We'll use redux-logger just as an example of adding another middleware
import logger from "redux-logger";

// And use redux-batched-subscribe as an example of adding enhancers
import { batchedSubscribe } from "redux-batched-subscribe";

import todosReducer from "./todos/todosReducer";
import visibilityReducer from "./visibility/visibilityReducer";

const reducer = {
  todos: todosReducer,
  visibility: visibilityReducer,
};

const preloadedState = {
  todos: [
    {
      text: "Eat food",
      completed: true,
    },
    {
      text: "Exercise",
      completed: false,
    },
  ],
  visibilityFilter: "SHOW_COMPLETED",
};

const debounceNotify = _.debounce((notify) => notify());

const store = configureStore({
  reducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== "production",
  preloadedState,
  enhancers: [batchedSubscribe(debounceNotify)],
});

// The store has been created with these options:
// - The slice reducers were automatically passed to combineReducers()
// - redux-thunk and redux-logger were added as middleware
// - The Redux DevTools Extension is disabled for production
// - The middleware, batched subscribe, and devtools enhancers were composed together
~~~
