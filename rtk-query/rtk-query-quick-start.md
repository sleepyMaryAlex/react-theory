# ?RTK query quick start

RTK Query is an advanced data fetching and caching tool, designed to simplify common cases for loading data in a web application. RTK Query itself is built on top of the Redux Toolkit core, and leverages RTK's APIs like `createSlice` and `createAsyncThunk` to implement its capabilities.

1. Create an API service

First, we'll create a service definition that queries the publicly available PokeAPI.

src/services/pokemon.ts
~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const pokemonApi = createApi({
  reducerPath: "pokemonApi",
  baseQuery: fetchBaseQuery({ baseUrl: "https://pokeapi.co/api/v2/" }),
  endpoints: (builder) => ({
    getPokemonByName: builder.query<any, string>({
      query: (name) => `pokemon/${name}`,
    }),
  }),
});

// Export hooks for usage in functional components, which are
// auto-generated based on the defined endpoints
export const { useGetPokemonByNameQuery } = pokemonApi;
~~~

With RTK Query, you usually define your entire API definition in one place. This is most likely different from what you see with other libraries, and there are several reasons for that. Our perspective is that it's much easier to keep track of how requests, cache invalidation, and general app configuration behave when they're all in one central location in comparison to having X number of custom hooks in different files throughout your application.

Typically, you should only have one API slice per base URL that your application needs to communicate with. For example, if your site fetches data from both `/api/posts` and `/api/users`, you would have a single API slice with `/api/` as the base URL, and separate endpoint definitions for posts and users. This allows you to effectively take advantage of automated re-fetching by defining tag relationships across endpoints.

2. Add the service to your store

An RTKQ service generates a "slice reducer" that should be included in the Redux root reducer, and a custom middleware that handles the data fetching. Both need to be added to the Redux store.

src/store.ts
~~~
import { configureStore } from "@reduxjs/toolkit";
import { setupListeners } from "@reduxjs/toolkit/query";
import { pokemonApi } from "./services/pokemon";

export const store = configureStore({
  reducer: {
    [pokemonApi.reducerPath]: pokemonApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(pokemonApi.middleware),
});

setupListeners(store.dispatch);
~~~

3. Wrap your application with the `Provider`

src/index.tsx
~~~
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import { store } from "./store";
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

4. Use the query in a component

Once a service has been defined, you can import the hooks to make a request.

src/App.tsx
~~~
import { useGetPokemonByNameQuery } from "./services/pokemon";

export default function App() {
  const { data, error, isLoading } = useGetPokemonByNameQuery("bulbasaur");

  return (
    <div className="App">
      {error ? (
        <>Oh no, there was an error</>
      ) : isLoading ? (
        <>Loading...</>
      ) : data ? (
        <>
          <h3>{data.species.name}</h3>
          <img src={data.sprites.front_shiny} alt={data.species.name} />
        </>
      ) : null}
    </div>
  );
}
~~~

You can always check `data`, `status`, and `error` to determine the right UI to render.

RTK Query ensures that any component that subscribes to the same query will always use the same data. RTK Query automatically de-dupes requests so you don't have to worry about checking in-flight requests and performance optimizations on your end.
