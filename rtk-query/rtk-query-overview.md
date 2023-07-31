# ?RTK Query Overview

RTK Query is a powerful data fetching and caching tool. It is designed to simplify common cases for loading data in a web application, eliminating the need to hand-write data fetching & caching logic yourself.

### Motivation

Web applications normally need to fetch data from a server in order to display it. They also usually need to make updates to that data, send those updates to the server, and keep the cached data on the client in sync with the data on the server. This is made more complicated by the need to implement other behaviors used in today's applications:

* Tracking loading state in order to show UI spinners
* Avoiding duplicate requests for the same data
* Optimistic updates to make the UI feel faster
* Managing cache lifetimes as the user interacts with the UI

RTK Query takes inspiration from other tools that have pioneered solutions for data fetching, but adds a unique approach to its API design:

* The data fetching and caching logic is built on top of Redux Toolkit's `createSlice` and `createAsyncThunk` APIs
* Because Redux Toolkit is UI-agnostic, RTK Query's functionality can be used with any UI layer
* API endpoints are defined ahead of time, including how to generate query parameters from arguments and transform responses for caching
* RTK Query can also generate React hooks that encapsulate the entire data fetching process, provide `data` and `isLoading` fields to components, and manage the lifetime of cached data as components mount and unmount
* RTK Query provides "cache entry lifecycle" options that enable use cases like streaming cache updates via websocket messages after fetching the initial data
* We have early working examples of code generation of API slices from OpenAPI and GraphQL schemas
* Finally, RTK Query is completely written in TypeScript, and is designed to provide an excellent TS usage experience

### APIs

RTK Query includes these APIs:

* `createApi()`: The core of RTK Query's functionality. It allows you to define a set of endpoints describe how to retrieve data from a series of endpoints, including configuration of how to fetch and transform that data. In most cases, you should use this once per app, with "one API slice per base URL" as a rule of thumb.
* `fetchBaseQuery()`: A small wrapper around `fetch` that aims to simplify requests. Intended as the recommended `baseQuery` to be used in `createApi` for the majority of users.
* `<ApiProvider />`: Can be used as a `Provider` if you do not already have a Redux store.
* `setupListeners()`: A utility used to enable `refetchOnMount` and `refetchOnReconnect` behaviors.

> Because RTK Query dispatches normal Redux actions as requests are processed, all actions are visible in the Redux DevTools.
