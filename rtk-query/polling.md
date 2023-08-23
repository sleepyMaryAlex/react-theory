# ?Polling

Polling gives you the ability to have a 'real-time' effect by causing a query to run at a specified interval. To enable polling for a query, pass a `pollingInterval` to the `useQuery` hook or action creator with an interval in milliseconds:

src/Pokemon.tsx
~~~
import * as React from "react";
import { useGetPokemonByNameQuery } from "./services/pokemon";

export const Pokemon = ({ name }: { name: string }) => {
  // Automatically refetch every 3s
  const { data, status, error, refetch } = useGetPokemonByNameQuery(name, {
    pollingInterval: 3000,
  });

  return <div>{data}</div>;
};
~~~
