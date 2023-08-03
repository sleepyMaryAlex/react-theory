# ?createEntityAdapter

A function that generates a set of prebuilt reducers and selectors for performing CRUD operations on a normalized state structure containing instances of a particular type of data object. These reducer functions may be passed as case reducers to `createReducer` and `createSlice`. They may also be used as "mutating" helper functions inside of `createReducer` and `createSlice`.

The methods generated by `createEntityAdapter` will all manipulate an "entity state" structure that looks like:

~~~
{
  // The unique IDs of each item. Must be strings or numbers
  ids: []
  // A lookup table mapping entity IDs to the corresponding entity objects
  entities: {
  }
}
~~~

Sample usage:

~~~
import {
  createEntityAdapter,
  createSlice,
  configureStore,
} from "@reduxjs/toolkit";

type Book = { bookId: string; title: string };

const booksAdapter = createEntityAdapter<Book>({
  // Assume IDs are stored in a field other than `book.id`
  selectId: (book) => book.bookId,
  // Keep the "all IDs" array sorted based on book titles
  sortComparer: (a, b) => a.title.localeCompare(b.title),
});

const booksSlice = createSlice({
  name: "books",
  initialState: booksAdapter.getInitialState(),
  reducers: {
    // Can pass adapter functions directly as case reducers.  Because we're passing this
    // as a value, `createSlice` will auto-generate the `bookAdded` action type / creator
    bookAdded: booksAdapter.addOne,
    booksReceived(state, action) {
      // Or, call them as "mutating" helpers in a case reducer
      booksAdapter.setAll(state, action.payload);
    },
  },
});

const store = configureStore({
  reducer: {
    books: booksSlice.reducer,
  },
});

type RootState = ReturnType<typeof store.getState>;

console.log(store.getState().books);
// { ids: [], entities: {} }

// Can create a set of memoized selectors based on the location of this entity state
const booksSelectors = booksAdapter.getSelectors<RootState>(
  (state) => state.books
);

const { bookAdded, booksReceived } = booksSlice.actions;

store.dispatch(bookAdded({ bookId: "a", title: "First" }));
console.log(store.getState().books);
// { ids: ['a'], entities: { a: {bookId: 'a', title: 'First'}} }
store.dispatch(
  booksReceived([
    { bookId: "b", title: "Book 2" },
    { bookId: "c", title: "Book 3" },
  ])
);

console.log(booksSelectors.selectIds(store.getState()));
// ['b', 'c']

// And then use the selectors to retrieve values
console.log(booksSelectors.selectAll(store.getState()));
// [{bookId: 'b', title: 'Book 2'}, {bookId: 'c', title: 'Book 3'}]

export default store;
~~~

### Parameters

`createEntityAdapter` accepts a single options object parameter, with two optional fields inside.

1. `selectId`. A function that accepts a single Entity instance, and returns the value of whatever unique ID field is inside.
2. `sortComparer`. A callback function that accepts two Entity instances, and should return a standard `Array.sort()` numeric result `(1, 0, -1)` to indicate their relative order for sorting.

Note that sorting only kicks in when state is changed via one of the CRUD functions below (for example, `addOne()`, `updateMany()`).

### Return Value

A "entity adapter" instance. An entity adapter is a plain JS object (not a class) containing the generated reducer functions, the original provided `selectId` and `sortComparer` callbacks, a method to generate an initial "entity state" value, and functions to generate a set of globalized and non-globalized memoized selector functions for this entity type.

### CRUD Functions

The primary content of an entity adapter is a set of generated reducer functions for adding, updating, and removing entity instances from an entity state object:

* `addOne`: accepts a single entity, and adds it if it's not already present.
* `addMany`: accepts an array of entities or an object in the shape of `Record<EntityId, T>`, and adds them if not already present.
* `setOne`: accepts a single entity and adds or replaces it
* `setMany`: accepts an array of entities or an object in the shape of `Record<EntityId, T>`, and adds or replaces them.
...

### getInitialState

Returns a new entity state object like `{ids: [], entities: {}}`. It accepts an optional object as an argument. The fields in that object will be merged into the returned initial state value.

### Selector Functions

The entity adapter will contain a `getSelectors()` function that returns a set of selectors that know how to read the contents of an entity state object:

* `selectIds`: returns the `state.ids` array.
* `selectEntities`: returns the `state.entities` lookup table.
* `selectAll`: maps over the `state.ids` array, and returns an array of entities in the same order.
* `selectTotal`: returns the total number of entities being stored in this state.
* `selectById`: given the state and an entity ID, returns the entity with that ID or `undefined`.