# ?Usage guide

### `configureStore`

The simplest way to use it is to just pass the root reducer function as a parameter named `reducer`:

~~~
import { configureStore } from "@reduxjs/toolkit";
import rootReducer from "./reducers";

const store = configureStore({
  reducer: rootReducer,
});

export default store;
~~~

You can also pass an object full of "slice reducers", and `configureStore` will call `combineReducers` for you:

~~~
import { configureStore } from "@reduxjs/toolkit";
import usersReducer from "./usersReducer";
import postsReducer from "./postsReducer";

const store = configureStore({
  reducer: {
    users: usersReducer,
    posts: postsReducer,
  },
});

export default store;
~~~

Note that this only works for one level of reducers. If you want to nest reducers, you'll need to call `combineReducers` yourself to handle the nesting.

### `createReducer`

In general, any Redux reducer that uses a `switch` statement can be converted to use `createReducer` directly. Each `case` in the switch becomes a key in the object passed to `createReducer`. Immutable update logic, like spreading objects or copying arrays, can probably be converted to direct "mutation". It's also fine to keep the immutable updates as-is and return the updated copies, too.

Here's some examples of how you can use `createReducer`. We'll start with a typical "todo list" reducer that uses switch statements and immutable updates:

~~~
function todosReducer(state = [], action) {
  switch (action.type) {
    case "ADD_TODO": {
      return state.concat(action.payload);
    }
    case "TOGGLE_TODO": {
      const { index } = action.payload;
      return state.map((todo, i) => {
        if (i !== index) return todo;

        return {
          ...todo,
          completed: !todo.completed,
        };
      });
    }
    case "REMOVE_TODO": {
      return state.filter((todo, i) => i !== action.payload.index);
    }
    default:
      return state;
  }
}
~~~

Notice that we specifically call `state.concat()` to return a copied array with the new todo entry, `state.map()` to return a copied array for the toggle case, and use the object spread operator to make a copy of the todo that needs to be updated.

With `createReducer`, we can shorten that example considerably:

~~~
const todosReducer = createReducer([], (builder) => {
  builder
    .addCase("ADD_TODO", (state, action) => {
      state.push(action.payload);
    })
    .addCase("TOGGLE_TODO", (state, action) => {
      const todo = state[action.payload.index];
      todo.completed = !todo.completed;
    })
    .addCase("REMOVE_TODO", (state, action) => {
      return state.filter((todo, i) => i !== action.payload.index);
    });
});
~~~

The ability to "mutate" the state is especially helpful when trying to update deeply nested state. This complex and painful code:

~~~
case "UPDATE_VALUE":
  return {
    ...state,
    first: {
      ...state.first,
      second: {
        ...state.first.second,
        [action.someId]: {
          ...state.first.second[action.someId],
          fourth: action.someValue
        }
      }
    }
  }
~~~

Can be simplified down to just:

~~~
updateValue(state, action) {
  const {someId, someValue} = action.payload;
  state.first.second[someId].fourth = someValue;
}
~~~

While the Redux Toolkit createReducer function can be really helpful, keep in mind that:

* The "mutative" code only works correctly inside of our `createReducer` function.
* Immer won't let you mix "mutating" the draft state and also returning a new state value.

### `createAction`

A typical action creator might look like:

~~~
function addTodo(text) {
  return {
    type: 'ADD_TODO',
    payload: { text },
  }
}
~~~

Writing action creators by hand can get tedious. Redux Toolkit provides a function called `createAction`, which simply generates an action creator that uses the given action type, and turns its argument into the `payload` field:

~~~
const addTodo = createAction('ADD_TODO')
addTodo({ text: 'Buy milk' })
// {type : "ADD_TODO", payload : {text : "Buy milk"}}
~~~

`createAction` overrides the `toString()` method on the action creators it generates. This means that the action creator itself can be used as the "action type" reference in some places, such as the keys provided to `builder.addCase` or the `createReducer` object notation.

~~~
const actionCreator = createAction("SOME_ACTION_TYPE");

console.log(actionCreator.toString());
// "SOME_ACTION_TYPE"

console.log(actionCreator.type);
// "SOME_ACTION_TYPE"

const reducer = createReducer({}, (builder) => {
  // actionCreator.toString() will automatically be called here
  // also, if you use TypeScript, the action type will be correctly inferred
  builder.addCase(actionCreator, (state, action) => {});
});
~~~

Unfortunately, the implicit conversion to a string doesn't happen for `switch` statements. If you want to use one of these action creators in a switch statement, you need to call `actionCreator.toString()` yourself:

~~~
const actionCreator = createAction("SOME_ACTION_TYPE");

const reducer = (state = {}, action) => {
  switch (action.type) {
    // CORRECT: this will work as expected
    case actionCreator.toString(): {
      break;
    }
    // CORRECT: this will also work right
    case actionCreator.type: {
      break;
    }
  }
};
~~~

If you are using Redux Toolkit with TypeScript, note that the TypeScript compiler may not accept the implicit `toString()` conversion when the action creator is used as an object key. In that case, you may need to either manually cast it to a string (`actionCreator as string`), or use the `.type` field as the key.

### `createSlice`

Redux Toolkit includes a `createSlice` function that will auto-generate the action types and action creators for you, based on the names of the reducer functions you provide.

~~~
const postsSlice = createSlice({
  name: 'posts',
  initialState: [],
  reducers: {
    createPost(state, action) {},
    updatePost(state, action) {},
    deletePost(state, action) {},
  },
})

const { actions, reducer } = postsSlice

export const { createPost, updatePost, deletePost } = actions

export default reducer;
~~~

`createSlice` looked at all of the functions that were defined in the `reducers` field, and for every "case reducer" function provided, generates an action creator that uses the name of the reducer as the action type itself. So, the `createPost` reducer became an action type of `"posts/createPost"`, and the `createPost()` action creator will return an action with that type.

Redux action types are not meant to be exclusive to a single slice. Conceptually, each slice reducer "owns" its own piece of the Redux state, but it should be able to listen to any action type and update its state appropriately. For example, many different slices might want to respond to a "user logged out" action by clearing data or resetting back to initial state values.

`createSlice` uses `createReducer` inside, so it's also safe to "mutate" state there as well.

### Middleware

The most common reason to use middleware is to allow different kinds of async logic to interact with the store. This allows you to write code that can dispatch actions and check the store state, while keeping that logic separate from your UI.

There are many kinds of async middleware for Redux, and each lets you write your logic using different syntax. The most common async middleware are:

* `redux-thunk`, which lets you write plain functions that may contain async logic directly.
* `redux-saga`, which uses generator functions that return descriptions of behavior so they can be executed by the middleware.
* `redux-observable`, which uses the RxJS observable library to create chains of functions that process actions.

The Redux Toolkit `configureStore` function automatically sets up the `thunk` middleware by default, so you can immediately start writing thunks as part of your application code.

### `createAsyncThunk`

~~~
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import { userAPI } from "./userAPI";

// First, create the thunk
const fetchUserById = createAsyncThunk(
  "users/fetchByIdStatus",
  async (userId, thunkAPI) => {
    const response = await userAPI.fetchById(userId);
    return response.data;
  }
);

// Then, handle actions in your reducers:
const usersSlice = createSlice({
  name: "users",
  initialState: { entities: [], loading: "idle" },
  reducers: {
    // standard reducer logic, with auto-generated action types per reducer
  },
  extraReducers: (builder) => {
    // Add reducers for additional action types here, and handle loading state as needed
    builder.addCase(fetchUserById.fulfilled, (state, action) => {
      // Add user to the state array
      state.entities.push(action.payload);
    });
  },
});

// Later, dispatch the thunk as needed in the app
dispatch(fetchUserById(123));
~~~
