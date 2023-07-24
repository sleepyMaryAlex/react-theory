# ?createListenerMiddleware

A Redux middleware that lets you define "listener" entries that contain an "effect" callback with additional logic, and a way to specify when that callback should run based on dispatched actions or state changes.

It's intended to be a lightweight alternative to more widely used Redux async middleware like sagas and observables. While similar to thunks in level of complexity and concept, it can be used to replicate some common saga usage patterns.

Conceptually, you can think of this as being similar to React's `useEffect` hook, except that it runs logic in response to Redux store updates instead of component props/state updates.

index.tsx
~~~
import { createRoot } from "react-dom/client";
import { Provider } from "react-redux";
import store from "./store";
import App from "./App";

const root = createRoot(document.getElementById("app-root") as HTMLElement);
root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
~~~

App.tsx
~~~
import { useSelector, useDispatch } from "react-redux";

export default function App() {
  const value = useSelector((state: number) => state);
  const dispatch = useDispatch();
  return (
    <div>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <span>{value}</span>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
    </div>
  );
}
~~~

reducer.ts
~~~
import { createReducer } from "@reduxjs/toolkit";

const reducer = createReducer(100, (builder) => {
  builder
    .addCase("INCREMENT", (state) => {
      console.log("updating...");
      return state + 1;
    })
    .addCase("DECREMENT", (state) => {
      console.log("updating...");
      return state - 1;
    });
});

export default reducer;
~~~

store.ts
~~~
import reducer from "./reducer";
import {
  configureStore,
  createListenerMiddleware,
  createAction,
  isAnyOf,
  Action,
} from "@reduxjs/toolkit";

const increment = createAction("INCREMENT");
const decrement = createAction("DECREMENT");

const listenerMiddleware = createListenerMiddleware();

const someTasks = (ms: number) =>
  new Promise((resolve) => setTimeout(resolve, ms));

const isIncrement = (action: Action) => {
  return action.type === "INCREMENT";
};

listenerMiddleware.startListening({
  matcher: isAnyOf(increment, decrement),
  effect: async (action, listenerApi) => {
    console.log("effect");
    if (await listenerApi.condition(isIncrement)) {
      console.log("is INCREMENT");
      await listenerApi.pause(someTasks(3000));
      console.log("paused for 3s");
      const r = await listenerApi.take(isIncrement, 2000);
      console.log(r);
    }
  },
});

export default configureStore({
  reducer,
  preloadedState: 0,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().prepend(listenerMiddleware.middleware),
});
~~~
