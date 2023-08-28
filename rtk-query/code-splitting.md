# ?Code Splitting

RTK Query makes it possible to trim down your initial bundle size by allowing you to inject additional endpoints after you've set up your initial service definition. This can be very beneficial for larger applications that may have many endpoints.

`injectEndpoints` accepts a collection of endpoints, as well as an optional `overrideExisting` parameter.

Calling `injectEndpoints` will inject the endpoints into the original API, but also give you that same API with correct types for these endpoints back. (Unfortunately, it cannot modify the types for the original definition.)

A typical approach would be to have one empty central API slice definition:

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

// initialize an empty api service that we'll inject endpoints into later as needed
export const emptySplitApi = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  endpoints: () => ({}),
});
~~~

And then inject the api endpoints in other files and export them from there - that way you will be sure to always import the endpoints in a way that they are definitely injected.

~~~
import { emptySplitApi } from "./emptySplitApi";

const extendedApi = emptySplitApi.injectEndpoints({
  endpoints: (build) => ({
    example: build.query({
      query: () => "test",
    }),
  }),
  overrideExisting: false,
});

export const { useExampleQuery } = extendedApi;
~~~
