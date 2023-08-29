# ?Automated Re-fetching

RTK Query uses a "cache tag" system to automate re-fetching for query endpoints that have data affected by mutation endpoints. This enables designing your API such that firing a specific mutation will cause a certain query endpoint to consider its cached data invalid, and re-fetch the data if there is an active subscription.

### Tags

Tags are defined in the `tagTypes` argument when defining an api. For example, in an application that has both `Posts` and `Users`, you might define `tagTypes: ['Post', 'User']` when calling `createApi`.

An individual `tag` has a `type`, represented as a `string` name, and an optional `id`, represented as a `string` or `number`. It can be represented as a plain string (such as `'Post'`), or an object in the shape `{type: string, id?: string|number}` (such as `[{type: 'Post', id: 1}]`).

### Providing tags

A query can have its cached data provide tags. Doing so determines which 'tag' is attached to the cached data returned by the query.

The `providesTags` argument can either be an array of `string` (such as `['Post']`), `{type: string, id?: string|number}` (such as `[{type: 'Post', id: 1}]`), or a callback that returns such an array. That function will be passed the result as the first argument, the response error as the second argument, and the argument originally passed into the `query` method as the third argument. Note that either the result or error arguments may be undefined based on whether the query was successful or not.

The example below declares that the `getPosts` `query` endpoint provides the `'Post'` tag to the cache, using the `providesTags` property for a `query` endpoint.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query";
import type { Post, User } from "./types";

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/",
  }),
  tagTypes: ["Post", "User"],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => "/posts",
      providesTags: ["Post"],
    }),
    getUsers: build.query<User[], void>({
      query: () => "/users",
      providesTags: ["User"],
    }),
    addPost: build.mutation<Post, Omit<Post, "id">>({
      query: (body) => ({
        url: "posts",
        method: "POST",
        body,
      }),
    }),
    editPost: build.mutation<Post, Partial<Post> & Pick<Post, "id">>({
      query: (body) => ({
        url: `post/${body.id}`,
        method: "POST",
        body,
      }),
    }),
  }),
});
~~~

For more granular control over the provided data, provided `tags` can have an associated `id`. This enables a distinction between 'any of a particular tag type', and 'a specific instance of a particular tag type'.

The example below declares that the provided posts are associated with particular IDs as determined by the result returned by the endpoint:

~~~
getPosts: build.query<Post[], void>({
  query: () => '/posts',
  providesTags: (result, error, arg) =>
    result
      ? [...result.map(({ id }) => ({ type: 'Post' as const, id })), 'Post']
      : ['Post'],
}),
~~~

### Invalidating tags

A mutation can invalidate specific cached data based on the tags. Doing so determines which cached data will be either refetched or removed from the cache.

The `invalidatesTags` argument can either be an array of `string` (such as `['Post']`), `{type: string, id?: string|number}` (such as `[{type: 'Post', id: 1}]`), or a callback that returns such an array. That function will be passed the result as the first argument, the response error as the second argument, and the argument originally passed into the `query` method as the third argument. Note that either the result or error arguments may be undefined based on whether the mutation was successful or not.

Each individual mutation endpoint can `invalidate` particular tags for existing cached data. Doing so enables a relationship between cached data from one or more query endpoints and the behaviour of one or more mutation endpoints.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query";
import type { Post, User } from "./types";

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/",
  }),
  tagTypes: ["Post", "User"],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => "/posts",
      providesTags: (result, error, arg) =>
        result
          ? [...result.map(({ id }) => ({ type: "Post" as const, id })), "Post"]
          : ["Post"],
    }),
    getUsers: build.query<User[], void>({
      query: () => "/users",
      providesTags: ["User"],
    }),
    addPost: build.mutation<Post, Omit<Post, "id">>({
      query: (body) => ({
        url: "post",
        method: "POST",
        body,
      }),
      invalidatesTags: ["Post"],
    }),
    editPost: build.mutation<Post, Partial<Post> & Pick<Post, "id">>({
      query: (body) => ({
        url: `post/${body.id}`,
        method: "POST",
        body,
      }),
      invalidatesTags: ["Post"],
    }),
  }),
});
~~~

For the example above, this tells RTK Query that after the `addPost` and/or `editPost` mutations are called and completed, any cache data supplied with the `'Post'` tag is no longer valid. If a component is currently subscribed to the cached data for a `'Post'` tag after the above mutations are called and complete, it will automatically re-fetch in order to retrieve up to date data from the server.

The example below declares that the `editPost` mutation invalidates a specific instance of a `Post` tag, using the ID passed in when calling the mutation function:

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query";
import type { Post, User } from "./types";

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: "/",
  }),
  tagTypes: ["Post", "User"],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => "/posts",
      providesTags: (result, error, arg) =>
        result
          ? [...result.map(({ id }) => ({ type: "Post" as const, id })), "Post"]
          : ["Post"],
    }),
    getUsers: build.query<User[], void>({
      query: () => "/users",
      providesTags: ["User"],
    }),
    addPost: build.mutation<Post, Omit<Post, "id">>({
      query: (body) => ({
        url: "post",
        method: "POST",
        body,
      }),
      invalidatesTags: ["Post"],
    }),
    editPost: build.mutation<Post, Partial<Post> & Pick<Post, "id">>({
      query: (body) => ({
        url: `post/${body.id}`,
        method: "POST",
        body,
      }),
      invalidatesTags: (result, error, arg) => [{ type: "Post", id: arg.id }],
    }),
  }),
});
~~~
