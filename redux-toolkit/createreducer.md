# ?createReducer

A utility that simplifies creating Redux reducer functions. It uses Immer internally to drastically simplify immutable update logic by writing "mutative" code in your reducers, and supports directly mapping specific action types to case reducer functions that will update the state when that action is dispatched.

Redux reducers are often implemented using a `switch` statement, with one `case` for every handled action type.

The `createReducer` helper streamlines the implementation of such reducers. It supports two different forms of defining case reducers to handle actions: a "builder callback" notation and a "map object" notation. Both are equivalent, but the "builder callback" notation is preferred.

With `createReducer`, your reducers instead look like:

~~~
import { createAction, createReducer } from "@reduxjs/toolkit";

interface CounterState {
  value: number;
}

const increment = createAction("counter/increment");
const decrement = createAction("counter/decrement");
const incrementByAmount = createAction<number>("counter/incrementByAmount");

const initialState = { value: 0 } as CounterState;

const counterReducer = createReducer(initialState, (builder) => {
  builder
    .addCase(increment, (state, action) => {
      state.value++;
    })
    .addCase(decrement, (state, action) => {
      state.value--;
    })
    .addCase(incrementByAmount, (state, action) => {
      state.value += action.payload;
    });
});
~~~

### Usage with the "Builder Callback" Notation

This overload accepts a callback function that receives a `builder` object as its argument. That builder provides `addCase`, `addMatcher` and `addDefaultCase` functions that may be called to define what actions this reducer will handle.

The recommended way of using `createReducer` is the builder callback notation, as it works best with TypeScript and most IDEs.

### Builder Methods

* `builder.addCase`

Adds a case reducer to handle a single exact action type.

All calls to `builder.addCase` must come before any calls to `builder.addMatcher` or `builder.addDefaultCase`.

* `builder.addMatcher`

Allows you to match your incoming actions against your own filter function instead of only the `action.type` property.

If multiple matcher reducers match, all of them will be executed in the order they were defined in - even if a case reducer already matched. All calls to `builder.addMatcher` must come after any calls to `builder.addCase` and before any calls to `builder.addDefaultCase`.

* `builder.addDefaultCase`

Adds a "default case" reducer that is executed if no case reducer and no matcher reducer was executed for this action.

~~~
import {
  createAction,
  createReducer,
  AnyAction,
  PayloadAction,
  configureStore,
} from "@reduxjs/toolkit";

export const increment = createAction<number>("increment");
export const decrement = createAction<number>("decrement");

function isActionWithNumberPayload(
  action: AnyAction
): action is PayloadAction<number> {
  return typeof action.payload === "number";
}

const reducer = createReducer(
  {
    counter: 0,
    sumOfNumberPayloads: 0,
    unhandledActions: 0,
  },
  (builder) => {
    builder
      .addCase(increment, (state, action) => {
        state.counter += action.payload;
        console.log(state.counter);
      })
      .addCase(decrement, (state, action) => {
        state.counter -= action.payload;
        console.log(state.counter);
      })
      .addMatcher(isActionWithNumberPayload, (state, action) => {
        console.log("is action with number payload");
      })
      .addDefaultCase((state, action) => {
        console.log("default");
      });
  }
);

const store = configureStore({
  reducer,
});

export type AppDispatch = typeof store.dispatch;
export type RootState = ReturnType<typeof store.getState>;

export default store;
~~~

### Direct State Mutation

To make things easier, `createReducer` uses immer to let you write reducers as if they were mutating the state directly. In reality, the reducer receives a proxy state that translates all mutations into equivalent copy operations.

Writing "mutating" reducers simplifies the code. It's shorter, there's less indirection, and it eliminates common mistakes made while spreading nested state. However, the use of Immer does add some "magic", and Immer has its own nuances in behavior. You should read through pitfalls mentioned in the immer docs . Most importantly, __you need to ensure that you either mutate the state argument or return a new state, but not both__. For example, the following reducer would throw an exception if a toggleTodo action is passed:

~~~
import { configureStore, createAction, createReducer } from "@reduxjs/toolkit";

interface Todo {
  text: string;
  completed: boolean;
}

const toggleTodo = createAction<number>("todos/toggle");

const todosReducer = createReducer([] as Todo[], (builder) => {
  builder.addCase(toggleTodo, (state, action) => {
    const index = action.payload;
    const todo = state[index];

    // This case reducer both mutates the passed-in state...
    todo.completed = !todo.completed;

    // ... and returns a new value. This will throw an
    // exception. In this example, the easiest fix is
    // to remove the `return` statement.
    return [...state.slice(0, index), todo, ...state.slice(index + 1)];
  });
});

const store = configureStore({
  reducer: todosReducer,
});

export type AppDispatch = typeof store.dispatch;
export type RootState = ReturnType<typeof store.getState>;

export default store;
~~~

### Multiple Case Reducer Execution

Originally, `createReducer` always matched a given action type to a single case reducer, and only that one case reducer would execute for a given action.

Using action matchers changes that behavior, as multiple matchers may handle a single action.

For any dispatched action, the behavior is:

* If there is an exact match for the action type, the corresponding case reducer will execute first
* Any matchers that return `true` will execute in the order they were defined
* If a default case reducer is provided, and no case or matcher reducers ran, the default case reducer will execute
* If no case or matcher reducers ran, the original existing state value will be returned unchanged

The executing reducers form a pipeline, and each of them will receive the output of the previous reducer:

~~~
import { createReducer } from "@reduxjs/toolkit";

const reducer = createReducer(0, (builder) => {
  builder
    .addCase("increment", (state) => state + 1)
    .addMatcher(
      (action) => action.type.startsWith("i"),
      (state) => state * 5
    )
    .addMatcher(
      (action) => action.type.endsWith("t"),
      (state) => state + 2
    );
});

console.log(reducer(0, { type: "increment" }));
// Returns 7, as the 'increment' case and both matchers all ran in sequence:
// - case 'increment": 0 => 1
// - matcher starts with 'i': 1 => 5
// - matcher ends with 't': 5 => 7
~~~

### Logging Draft State Values

It's very common for a developer to call `console.log(state)` during the development process. However, browsers display Proxies in a format that is hard to read, which can make console logging of Immer-based state difficult.

When using either `createSlice` or `createReducer`, you may use the current utility that we re-export from the `immer` library. This utility creates a separate plain copy of the current Immer `Draft` state value, which can then be logged for viewing as normal.

~~~
import { createSlice, current } from "@reduxjs/toolkit";

const slice = createSlice({
  name: "todos",
  initialState: [{ id: 1, title: "Example todo" }],
  reducers: {
    addTodo: (state, action) => {
      console.log("before", current(state));
      state.push(action.payload);
      console.log("after", current(state));
    },
  },
});
~~~
