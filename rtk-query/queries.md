# ?Queries

This is the most common use case for RTK Query. A query operation can be performed with any data fetching library of your choice, but the general recommendation is that you only use queries for requests that retrieve data. For anything that alters data on the server or will possibly invalidate the cache, you should use a Mutation.

### Defining Query Endpoints

Query endpoints are defined by returning an object inside the `endpoints` section of `createApi`, and defining the fields using the `builder.query()` method.

Query endpoints should define either a `query` callback that constructs the URL (including any URL query params), or a `queryFn` callback that may do arbitrary async logic and return a result.

If the `query` callback needs additional data to generate the URL, it should be written to take a single argument. If you need to pass in multiple parameters, pass them formatted as a single "options object".

Query endpoints may also modify the response contents before the result is cached, define "tags" to identify cache invalidation, and provide cache entry lifecycle callbacks to run additional logic as cache entries are added and removed.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query";
import type { Post } from "./types";

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/",
  }),
  tagTypes: ["Post"],
  endpoints: (build) => ({
    getPost: build.query<Post, number>({
      // note: an optional `queryFn` may be used in place of `query`
      query: (id) => ({ url: `post/${id}` }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response: { data: Post }, meta, arg) => response.data,
      // Pick out errors and prevent nested properties in a hook or selector
      transformErrorResponse: (
        response: { status: string | number },
        meta,
        arg
      ) => response.status,
      providesTags: (result, error, id) => [{ type: "Post", id }],
      // The 2nd parameter is the destructured `QueryLifecycleApi`
      async onQueryStarted(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          queryFulfilled,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
      // The 2nd parameter is the destructured `QueryCacheLifecycleApi`
      async onCacheEntryAdded(
        arg,
        {
          dispatch,
          getState,
          extra,
          requestId,
          cacheEntryRemoved,
          cacheDataLoaded,
          getCacheEntry,
          updateCachedData,
        }
      ) {},
    }),
  }),
});
~~~

### Performing Queries with React Hooks

If you're using React Hooks, RTK Query does a few additional things for you. The primary benefit is that you get a render-optimized hook that allows you to have 'background fetching' as well as derived booleans for convenience.

Hooks are automatically generated based on the name of the endpoint in the service definition. An `endpoint` field with `getPost: builder.query()` will generate a hook named `useGetPostQuery`.

#### Hook types

There are 5 query-related hooks:

1. `useQuery`

Composes `useQuerySubscription` and `useQueryState` and is the primary hook. Automatically triggers fetches of data from an endpoint, 'subscribes' the component to the cached data, and reads the request status and cached data from the Redux store.

2. `useQuerySubscription`

Returns a `refetch` function and accepts all hook options. Automatically triggers fetches of data from an endpoint, and 'subscribes' the component to the cached data.

3. `useQueryState`

Returns the query state and accepts `skip` and `selectFromResult`. Reads the request status and cached data from the Redux store.

4. `useLazyQuery`

Returns a tuple with a `trigger` function, the query result, and last promise info. Similar to `useQuery`, but with manual control over when the data fetching occurs. Note: the `trigger` function takes a second argument of `preferCacheValue?: boolean` in the event you want to skip making a request if cached data already exists.

5. `useLazyQuerySubscription`

Returns a tuple with a `trigger` function, and last promise info. Similar to `useQuerySubscription`, but with manual control over when the data fetching occurs. Note: the `trigger` function takes a second argument of `preferCacheValue?: boolean` in the event you want to skip making a request if cached data already exists.

In practice, the standard `useQuery`-based hooks such as `useGetPostQuery` will be the primary hooks used in your application, but the other hooks are available for specific use cases.

The query hooks expect two parameters: `(queryArg?, queryOptions?)`.

The `queryArg` param will be passed through to the underlying query callback to generate the URL.

The `queryOptions` object accepts several additional parameters that can be used to control the behavior of the data fetching:

* `skip` - Allows a query to 'skip' running for that render. Defaults to false
* `pollingInterval` - Allows a query to automatically refetch on a provided interval, specified in milliseconds. Defaults to 0 (off)
* `selectFromResult` - Allows altering the returned value of the hook to obtain a subset of the result, render-optimized for the returned subset.
* `refetchOnMountOrArgChange` - Allows forcing the query to always refetch on mount (when true is provided). Allows forcing the query to refetch if enough time (in seconds) has passed since the last query for the same cache (when a number is provided). Defaults to false
* `refetchOnFocus` - Allows forcing the query to refetch when the browser window regains focus. Defaults to false
* `refetchOnReconnect` - Allows forcing the query to refetch when regaining a network connection. Defaults to false

The query hook returns an object containing properties such as the latest data for the query request, as well as status booleans for the current request lifecycle state. Below are some of the most frequently used properties.

* `data` - The latest returned result regardless of hook arg, if present.
* `currentData` - The latest returned result for the current hook arg, if present.
* `error` - The error result if present.
* `isUninitialized` - When `true`, indicates that the query has not started yet.
* `isLoading` - When `true`, indicates that the query is currently loading for the first time, and has no data yet. This will be `true` for the first request fired off, but not for subsequent requests.
* `isFetching` - When `true`, indicates that the query is currently fetching, but might have data from an earlier request. This will be `true` for both the first request fired off, as well as subsequent requests.
* `isSuccess` - When `true`, indicates that the query has data from a successful request.
* `isError` - When `true`, indicates that the query is in an `error` state.
* `refetch` - A function to force refetch the query.

~~~
const {
  data: post,
  isFetching,
  isLoading,
} = useGetPostQuery(id, {
  pollingInterval: 3000,
  refetchOnMountOrArgChange: true,
  skip: false,
})
~~~

### Selecting data from a query result

Sometimes you may have a parent component that is subscribed to a query, and then in a child component you want to pick an item from that query. In most cases you don't want to perform an additional request for a `getItemById`-type query when you know that you already have the result.

`selectFromResult` allows you to get a specific segment from a query result in a performant manner.

~~~
function PostsList() {
  const { data: posts } = api.useGetPostsQuery();

  return (
    <ul>
      {posts?.data?.map((post) => (
        <PostById key={post.id} id={post.id} />
      ))}
    </ul>
  );
}

function PostById({ id }: { id: number }) {
  // Will select the post with the given id, and will only rerender if the given post's data changes
  const { post } = api.useGetPostsQuery(undefined, {
    selectFromResult: ({ data }) => ({
      post: data?.find((post) => post.id === id),
    }),
  });

  return <li>{post?.name}</li>;
}
~~~
