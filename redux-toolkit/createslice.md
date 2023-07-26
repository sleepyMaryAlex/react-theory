# ?createSlice

A function that accepts an initial state, an object of reducer functions, and a "slice name", and automatically generates action creators and action types that correspond to the reducers and state.

Internally, it uses `createAction` and `createReducer`, so you may also use Immer to write "mutating" immutable updates:

~~~
import { createSlice } from "@reduxjs/toolkit";
import type { PayloadAction } from "@reduxjs/toolkit";

interface CounterState {
  value: number;
}

const initialState = { value: 0 } as CounterState;

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment(state) {
      state.value++;
    },
    decrement(state) {
      state.value--;
    },
    incrementByAmount(state, action: PayloadAction<number>) {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
~~~

### Parameters

`createSlice` accepts a single configuration object parameter, with the following options:

1. `initialState`. The initial state value for this slice of state.
2. `name`. A string name for this slice of state. Generated action type constants will use this as a prefix.
3. `reducers`. An object containing Redux "case reducer" functions (functions intended to handle a specific action type, equivalent to a single case statement in a switch).

The keys in the object will be used to generate string action type constants, and these will show up in the Redux DevTools Extension when they are dispatched. Also, if any other part of the application happens to dispatch an action with the exact same type string, the corresponding reducer will be run. Therefore, you should give the functions descriptive names.

This object will be passed to `createReducer`, so the reducers may safely "mutate" the state they are given.

If you need to customize the creation of the payload value of an action creator by means of a `prepare callback`, the value of the appropriate field of the reducers argument object should be an object instead of a function. This object must contain two properties: `reducer` and `prepare`. The value of the `reducer` field should be the case reducer function while the value of the `prepare` field should be the prepare callback function:

~~~
import { configureStore } from "@reduxjs/toolkit";

import { createSlice } from "@reduxjs/toolkit";
import type { PayloadAction } from "@reduxjs/toolkit";

import { nanoid } from "@reduxjs/toolkit";

interface Item {
  id: string;
  text: string;
}

const todosSlice = createSlice({
  name: "todos",
  initialState: [] as Item[],
  reducers: {
    addTodo: {
      reducer: (state, action: PayloadAction<Item>) => {
        state.push(action.payload);
      },
      prepare: (text: string) => {
        const id = nanoid();
        return { payload: { id, text } };
      },
    },
  },
});

export const { addTodo } = todosSlice.actions;

const store = configureStore({
  reducer: todosSlice.reducer,
});

export type AppDispatch = typeof store.dispatch;
export type RootState = ReturnType<typeof store.getState>;

export default store;
~~~

> `nanoid` - a tiny, secure, URL-friendly, unique string ID generator for JavaScript.

### extraReducers

One of the key concepts of Redux is that each slice reducer "owns" its slice of state, and that many slice reducers can independently respond to the same action type. `extraReducers` allows `createSlice` to respond to other action types besides the types it has generated.

As case reducers specified with `extraReducers` are meant to reference "external" actions, they will not have actions generated in `slice.actions`.

As with reducers, these case reducers will also be passed to `createReducer` and may "mutate" their state safely.

If two fields from `reducers` and `extraReducers` happen to end up with the same action type string, the function from `reducers` will be used to handle that action type.

### The extraReducers "builder callback" notation

This builder notation is the only way to add matcher reducers and default case reducers to your slice.

~~~
import { createAction, createSlice, Action, AnyAction } from "@reduxjs/toolkit";
const incrementBy = createAction<number>("incrementBy");
const decrement = createAction("decrement");

interface RejectedAction extends Action {
  error: Error;
}

function isRejectedAction(action: AnyAction): action is RejectedAction {
  return action.type.endsWith("rejected");
}

createSlice({
  name: "counter",
  initialState: 0,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(incrementBy, (state, action) => {
        // action is inferred correctly here if using TS
      })
      // You can chain calls, or have separate `builder.addCase()` lines each time
      .addCase(decrement, (state, action) => {})
      // You can match a range of action types
      .addMatcher(
        isRejectedAction,
        // `action` will be inferred as a RejectedAction due to isRejectedAction being defined as a type guard
        (state, action) => {}
      )
      // and provide a default case if no other handlers matched
      .addDefaultCase((state, action) => {});
  },
});
~~~

We recommend using this API as it has better TypeScript support (and thus, IDE autocomplete even for JavaScript users), as it will correctly infer the action type in the reducer based on the provided action creator. It's particularly useful for working with actions produced by `createAction` and `createAsyncThunk`.

### Return Value

`createSlice` will return an object that looks like:

~~~
{
  name : string,
  reducer : ReducerFunction,
  actions : Record<string, ActionCreator>,
  caseReducers: Record<string, CaseReducer>.
  getInitialState: () => State
}
~~~
