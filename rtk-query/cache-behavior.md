# ?Cache Behavior

A key feature of RTK Query is its management of cached data. When data is fetched from the server, RTK Query will store the data in the Redux store as a 'cache'. When an additional request is performed for the same data, RTK Query will provide the existing cached data rather than sending an additional request to the server.

### Default Cache Behavior

When a request is attempted, if the data already exists in the cache, then that data is served and no new request is sent to the server. Otherwise, if the data does not exist in the cache, then a new request is sent, and the returned response is stored in the cache.

Subscriptions are reference-counted. Additional subscriptions that ask for the same endpoint+params increment the reference count. As long as there is an active 'subscription' to the data (e.g. if a component is mounted that calls a `useQuery` hook for the endpoint), then the data will remain in the cache. Once the subscription is removed (e.g. when last component subscribed to the data unmounts), after an amount of time (default 60 seconds), the data will be removed from the cache. The expiration time can be configured with the `keepUnusedDataFor` property for the API definition as a whole, as well as on a per-endpoint basis.

### Reducing subscription time with `keepUnusedDataFor`

Providing a value to `keepUnusedDataFor` as a number in seconds specifies how long the data should be kept in the cache after the subscriber reference count reaches zero.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import type { Post } from "./types";

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  // global configuration for the api
  keepUnusedDataFor: 30,
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], number>({
      query: () => `posts`,
      // configuration for an individual endpoint, overriding the api setting
      keepUnusedDataFor: 5,
    }),
  }),
});
~~~

### Re-fetching on demand with `refetch`/`initiate`

In order to achieve complete granular control over re-fetching data, you can use the `refetch` function returned as a result property from a `useQuery` or `useQuerySubscription` hook.

Calling the `refetch` function will force refetch the associated query.

Alternatively, you can dispatch the `initiate` thunk action for an endpoint, passing the option `forceRefetch: true` to the thunk action creator for the same effect.

~~~
import { useDispatch } from "react-redux";
import { useGetPostsQuery } from "./api";

const Component = () => {
  const dispatch = useDispatch();
  const { data, refetch } = useGetPostsQuery({ count: 5 });

  function handleRefetchOne() {
    // force re-fetches the data
    refetch();
  }

  function handleRefetchTwo() {
    // has the same effect as `refetch` for the associated query
    dispatch(
      api.endpoints.getPosts.initiate(
        { count: 5 },
        { subscribe: false, forceRefetch: true }
      )
    );
  }

  return (
    <div>
      <button onClick={handleRefetchOne}>Force re-fetch 1</button>
      <button onClick={handleRefetchTwo}>Force re-fetch 2</button>
    </div>
  );
};
~~~

### Encouraging re-fetching with `refetchOnMountOrArgChange`

Queries can be encouraged to re-fetch more frequently than usual via the `refetchOnMountOrArgChange` property.

`refetchOnMountOrArgChange` accepts either a boolean value, or a number as time in seconds.

Passing `false` (the default value) for this property will use the default behavior described above.

Passing `true` for this property will cause the endpoint to always refetch when a new subscriber to the query is added.

Configuring re-fetching on subscription if data exceeds a given time:

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import type { Post } from "./types";

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  // global configuration for the api
  refetchOnMountOrArgChange: 30,
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], number>({
      query: () => `posts`,
    }),
  }),
});
~~~

Forcing refetch on component mount:

~~~
import { useGetPostsQuery } from "./api";

const Component = () => {
  const { data } = useGetPostsQuery(
    { count: 5 },
    // this overrules the api definition setting,
    // forcing the query to always fetch when this component is mounted
    { refetchOnMountOrArgChange: true }
  );

  return <div>...</div>;
};
~~~

### Re-fetching on window focus with `refetchOnFocus`

The `refetchOnFocus` option allows you to control whether RTK Query will try to refetch all subscribed queries after the application window regains focus.

If you specify this option alongside `skip: true`, this will not be evaluated until skip is false.

Note that this requires `setupListeners` to have been called.

src/services/api.ts
~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import type { Post } from "./types";

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  // global configuration for the api
  refetchOnFocus: true,
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], number>({
      query: () => `posts`,
    }),
  }),
});
~~~

src/store.ts
~~~
// ...
setupListeners(store.dispatch);
~~~

### Re-fetching on network reconnection with `refetchOnReconnect`

The `refetchOnReconnect` option on `createApi` allows you to control whether RTK Query will try to refetch all subscribed queries after regaining a network connection.

If you specify this option alongside `skip: true`, this will not be evaluated until skip is false.

Note that this requires setupListeners to have been called.

src/services/api.ts
~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import type { Post } from "./types";

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  // global configuration for the api
  refetchOnReconnect: true,
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], number>({
      query: () => `posts`,
    }),
  }),
});
~~~

src/store.ts
~~~
// ...
setupListeners(store.dispatch);
~~~

### No Normalized or De-duplicated Cache

RTK Query deliberately does not implement a cache that would deduplicate identical items across multiple requests.

As an example, say that we have an API slice with getTodos and getTodo endpoints, and our components make the following queries:

* `getTodos()`
* `getTodos({filter: 'odd'})`
* `getTodo({id: 1})`

Each of these query results would include a Todo object that looks like `{ id: 1 }`.

In a fully normalized de-duplicating cache, only a single copy of this Todo object would be stored. However, RTK Query saves each query result independently in the cache. So, this would result in three separate copies of this Todo being cached in the Redux store.
