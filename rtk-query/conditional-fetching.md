# ?Conditional Fetching

Query hooks automatically begin fetching data as soon as the component is mounted. But, there are use cases where you may want to delay fetching data until some condition becomes true. RTK Query supports conditional fetching to enable that behavior.

If you want to prevent a query from automatically running, you can use the `skip` parameter in a hook.

~~~
const Pokemon = ({ name, skip }: { name: string; skip: boolean }) => {
  const { data, error, status } = useGetPokemonByNameQuery(name, {
    skip,
  });

  return (
    <div>
      {name} - {status}
    </div>
  );
};
~~~

When `skip` is `true` (or `skipToken` is passed in as `arg`):

* If the query has cached data:
  * The cached data will not be used on the initial load, and will ignore updates from any identical query until the `skip` condition is removed
  * The query will have a status of `uninitialized`
  * If `skip: false` is set after the initial load, the cached result will be used
* If the query does not have cached data:
  * The query will have a status of `uninitialized`
  * The query will not exist in the state when viewed with the dev tools
  * The query will not automatically fetch on mount
  * The query will not automatically run when additional components with the same query are added that do run
