# ?createAction

A helper function for defining a Redux action type and creator.

`function createAction(type, prepareAction?)`

The usual way to define an action in Redux is to separately declare an action type constant and an action creator function for constructing actions of that type.

~~~
const INCREMENT = "counter/increment";

function increment(amount: number) {
  return {
    type: INCREMENT,
    payload: amount,
  };
}

const action = increment(3);
~~~

The `createAction` helper combines these two declarations into one. It takes an action type and returns an action creator for that type. The action creator can be called either without arguments or with a `payload` to be attached to the action. Also, the action creator overrides `toString()` so that the action type becomes its string representation.

~~~
import { createAction } from "@reduxjs/toolkit";

const increment = createAction<number | undefined>("counter/increment");

let action = increment();
// { type: 'counter/increment' }

action = increment(3);
// returns { type: 'counter/increment', payload: 3 }

console.log(increment.toString());
// 'counter/increment'

console.log(`The action type is: ${increment}`);
// 'The action type is: counter/increment'
~~~

### Using Prepare Callbacks to Customize Action Contents

By default, the generated action creators accept a single argument, which becomes `action.payload`. This requires the caller to construct the entire payload correctly and pass it in.

In many cases, you may want to write additional logic to customize the creation of the `payload` value, such as accepting multiple parameters for the action creator, generating a random ID, or getting the current timestamp. To do this, `createAction` accepts an optional second argument: a "prepare callback" that will be used to construct the payload value.

~~~
import { createAction, nanoid } from "@reduxjs/toolkit";

const addTodo = createAction("todos/add", function prepare(text: string) {
  return {
    payload: {
      text,
      id: nanoid(),
      createdAt: new Date().toISOString(),
    },
  };
});

console.log(addTodo("Write more docs"));
/**
 * {
 *   type: 'todos/add',
 *   payload: {
 *     text: 'Write more docs',
 *     id: '4AJvwMSWEHCchcWYga3dj',
 *     createdAt: '2019-10-03T07:53:36.581Z'
 *   }
 * }
 **/
~~~

### Usage with `createReducer()`

Because of their `toString()` override, action creators returned by `createAction()` can be used directly as keys for the case reducers passed to `createReducer()`.

~~~
const increment = createAction<number>("counter/increment");
const decrement = createAction<number>("counter/decrement");

const counterReducer = createReducer(0, (builder) => {
  builder.addCase(increment, (state, action) => state + action.payload);
  builder.addCase(decrement, (state, action) => state - action.payload);
});
~~~
