# ?Mutations

Mutations are used to send data updates to the server and apply the changes to the local cache. Mutations can also invalidate cached data and force re-fetches.

### Defining Mutation Endpoints

Mutation endpoints are defined by returning an object inside the endpoints section of `createApi`, and defining the fields using the `build.mutation()` method.

Mutation endpoints should define either a `query` callback that constructs the URL (including any URL query params), or a `queryFn` callback that may do arbitrary async logic and return a result. The `query` callback may also return an object containing the URL, the HTTP method to use and a request body.

If the `query` callback needs additional data to generate the URL, it should be written to take a single argument. If you need to pass in multiple parameters, pass them formatted as a single "options object".

Mutation endpoints may also modify the response contents before the result is cached, define "tags" to identify cache invalidation, and provide cache entry lifecycle callbacks to run additional logic as cache entries are added and removed.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query";
import type { Post } from "./types";

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/",
  }),
  tagTypes: ["Post"],
  endpoints: (build) => ({
    updatePost: build.mutation<Post, Partial<Post> & Pick<Post, "id">>({
      // note: an optional `queryFn` may be used in place of `query`
      query: ({ id, ...patch }) => ({
        url: `post/${id}`,
        method: "PATCH",
        body: patch,
      }),
      // Pick out data and prevent nested properties in a hook or selector
      transformResponse: (response: { data: Post }, meta, arg) => response.data,
      // Pick out errors and prevent nested properties in a hook or selector
      transformErrorResponse: (
        response: { status: string | number },
        meta,
        arg
      ) => response.status,
      invalidatesTags: ["Post"],
      // onQueryStarted is useful for optimistic updates
      // The 2nd parameter is the destructured `MutationLifecycleApi`
      async onQueryStarted(
        arg,
        { dispatch, getState, queryFulfilled, requestId, extra, getCacheEntry }
      ) {},
      // The 2nd parameter is the destructured `MutationCacheLifecycleApi`
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
        }
      ) {},
    }),
  }),
});
~~~

### Performing Mutations with React Hooks

Unlike `useQuery`, `useMutation` returns a tuple. The first item in the tuple is the "trigger" function and the second element contains an object with `status`, `error`, and `data`.

Unlike the `useQuery` hook, the `useMutation` hook doesn't execute automatically. To run a mutation you have to call the trigger function returned as the first tuple value from the hook.

Below are some of the most frequently used properties on the "mutation result" object.

* `data` - The data returned from the latest trigger response, if present. If subsequent triggers from the same hook instance are called, this will return undefined until the new data is received. Consider component level caching if the previous response data is required for a smooth transition to new data.
* `error` - The error result if present.
* `isUninitialized` - When `true`, indicates that the mutation has not been fired yet.
* `isLoading` - When `true`, indicates that the mutation has been fired and is awaiting a response.
* `isSuccess` - When `true`, indicates that the last mutation fired has data from a successful request.
* `isError` - When `true`, indicates that the last mutation fired resulted in an error state.
* `reset` - A method to reset the hook back to it's original state and remove the current result from the cache.

### Shared Mutation Results

By default, separate instances of a `useMutation` hook are not inherently related to each other. Triggering one instance will not affect the result for a separate instance. This applies regardless of whether the hooks are called within the same component, or different components.

~~~
export const ComponentOne = () => {
  // Triggering `updatePostOne` will affect the result in this component,
  // but not the result in `ComponentTwo`, and vice-versa
  const [updatePost, result] = useUpdatePostMutation();

  return <div>...</div>;
};

export const ComponentTwo = () => {
  const [updatePost, result] = useUpdatePostMutation();

  return <div>...</div>;
};
~~~

RTK Query provides an option to share results across mutation hook instances using the `fixedCacheKey` option.

~~~
export const ComponentOne = () => {
  // Triggering `updatePostOne` will affect the result in both this component,
  // but as well as the result in `ComponentTwo`, and vice-versa
  const [updatePost, result] = useUpdatePostMutation({
    fixedCacheKey: "shared-update-post",
  });

  return <div>...</div>;
};

export const ComponentTwo = () => {
  const [updatePost, result] = useUpdatePostMutation({
    fixedCacheKey: "shared-update-post",
  });

  return <div>...</div>;
};
~~~
