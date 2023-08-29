# ?Streaming Updates

RTK Query gives you the ability to receive streaming updates for persistent queries. This enables a query to establish an ongoing connection to the server (typically using WebSockets), and apply updates to the cached data as additional information is received from the server.

Streaming updates can be used to enable the API to receive real-time updates to the back-end data, such as new entries being created, or important properties being updated.

To enable streaming updates for a query, pass the asynchronous `onCacheEntryAdded` function to the query, including the logic for how to update the query when streamed data is received.

### Using the `onCacheEntryAdded` Lifecycle

The `onCacheEntryAdded` lifecycle callback lets you write arbitrary async logic that will be executed after a new cache entry is added to the RTK Query cache (ie, after a component has created a new subscription to a given endpoint+params combination).

`onCacheEntryAdded` will be called with two arguments: the `arg` that was passed to the subscription, and an options object containing "lifecycle promises" and utility functions. You can use these to write sequenced logic that waits for data to be added, initiates server connections, applies partial updates, and cleans up the connection when the query subscription is removed.

Typically, you will `await cacheDataLoaded` to determine when the first data has been fetched, then use the `updateCacheData` utility to apply streaming updates as messages are received. `updateCacheData` is an Immer-powered callback that receives a `draft` of the current cache value. You may "mutate" the draft value to update it as needed based on the received values. RTK Query will then dispatch an action that applies a diffed patch based on those changes.

Finally, you can `await cacheEntryRemoved` to know when to clean up any server connections.

~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import { isMessage } from "./schemaValidators";

export type Channel = "redux" | "general";

export interface Message {
  id: number;
  channel: Channel;
  userName: string;
  text: string;
}

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  endpoints: (build) => ({
    getMessages: build.query<Message[], Channel>({
      query: (channel) => `messages/${channel}`,
      async onCacheEntryAdded(
        arg,
        { updateCachedData, cacheDataLoaded, cacheEntryRemoved }
      ) {
        // create a websocket connection when the cache subscription starts
        const ws = new WebSocket("ws://localhost:8080");
        try {
          // wait for the initial query to resolve before proceeding
          await cacheDataLoaded;

          // when data is received from the socket connection to the server,
          // if it is a message and for the appropriate channel,
          // update our query result with the received message
          const listener = (event: MessageEvent) => {
            const data = JSON.parse(event.data);
            if (!isMessage(data) || data.channel !== arg) return;

            updateCachedData((draft) => {
              draft.push(data);
            });
          };

          ws.addEventListener("message", listener);
        } catch {
          // no-op in case `cacheEntryRemoved` resolves before `cacheDataLoaded`,
          // in which case `cacheDataLoaded` will throw
        }
        // cacheEntryRemoved will resolve when the cache subscription is no longer active
        await cacheEntryRemoved;
        // perform cleanup steps once the `cacheEntryRemoved` promise resolves
        ws.close();
      },
    }),
  }),
});

export const { useGetMessagesQuery } = api;
~~~
