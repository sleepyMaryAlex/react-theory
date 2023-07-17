# ?Quick start

1. Install Redux Toolkit and React-Redux

~~~
npm install @reduxjs/toolkit react-redux
~~~

2. Create a Redux Store

Create a file named `src/app/store.ts`.

~~~
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
~~~

`configureStore` creates a Redux store, and also automatically configure the Redux DevTools extension so that you can inspect the store while developing.

3. Provide the Redux Store to React

Once the store is created, we can make it available to our React components by putting a React-Redux `<Provider>` around our application in `src/index.tsx`.

~~~
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import { store } from "./app/store";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(
  document.getElementById("app-root") as HTMLElement
);
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
~~~

4. Create a Redux State Slice

Add a new file named `src/features/counter/counterSlice.ts`.

Creating a slice requires a string name to identify the slice, an initial state value, and one or more reducer functions to define how the state can be updated. Once a slice is created, we can export the generated Redux action creators and the reducer function for the whole slice.

~~~
import { createSlice } from "@reduxjs/toolkit";
import type { PayloadAction } from "@reduxjs/toolkit";
import { RootState } from "app/store";

export interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0,
};

export const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export const selectCount = (state: RootState) => state.counter.value;

export default counterSlice.reducer;
~~~

Redux requires that we write all state updates immutably, by making copies of data and updating the copies. However, Redux Toolkit's `createSlice` and `createReducer` APIs use Immer inside to allow us to write "mutating" update logic that becomes correct immutable updates.

Each slice file should define a type for its initial state value, so that `createSlice` can correctly infer the type of `state` in each case reducer.

All generated actions should be defined using the `PayloadAction<T>` type from Redux Toolkit, which takes the type of the `action.payload` field as its generic argument.

You can safely import the `RootState` type from the store file here. It's a circular import, but the TypeScript compiler can correctly handle that for types. This may be needed for use cases like writing selector functions.

5. Use Redux State and Actions in React Components

Now we can use the React-Redux hooks to let React components interact with the Redux store. We can read data from the store with useSelector, and dispatch actions using useDispatch.

Create a `src/features/counter/Counter.tsx` file with a `<Counter>` component inside, then import that component into `App.tsx` and render it inside of `<App>`.

Counter.tsx
~~~
import { decrement, increment, selectCount } from "./counterSlice";
import { useAppDispatch, useAppSelector } from "app/hooks";

export function Counter() {
  const count = useAppSelector(selectCount);
  const dispatch = useAppDispatch();

  return (
    <div>
      <div>
        <button
          aria-label="Increment value"
          onClick={() => dispatch(increment())}
        >
          Increment
        </button>
        <span>{count}</span>
        <button
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())}
        >
          Decrement
        </button>
      </div>
    </div>
  );
}
~~~

App.tsx
~~~
import { Counter } from "features/counter/Counter";

export default function App() {
  return <Counter />;
}
~~~

While it's possible to import the `RootState` and `AppDispatch` types into each component, it's better to create typed versions of the `useDispatch` and `useSelector` hooks for usage in your application.

app/hooks.ts
~~~
import { useDispatch, useSelector } from "react-redux";
import type { TypedUseSelectorHook } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
~~~
