# ?Prefetching

The goal of prefetching is to make data fetch before the user navigates to a page or attempts to load some known content.

There are a handful of situations that you may want to do this, but some very common use cases are:

1. User hovers over a navigation element.
2. User hovers over a list element that is a link.
3. User hovers over a next pagination button.
4. User navigates to a page and you know that some components down the tree will require said data. This way, you can prevent fetching waterfalls.

### Prefetching with React Hooks

Similar to the `useMutation` hook, the `usePrefetch` hook will not run automatically — it returns a "trigger function" that can be used to initiate the behavior.

It accepts two arguments: the first is the key of a query action that you defined in your API service, and the second is an object of two optional parameters:

* `ifOlderThan` - (default: `false` | `number`) - number is value in seconds

If specified, it will only run the query if the difference between `new Date()` and the last `fulfilledTimeStamp` is greater than the given value

* `force`

If `force: true`, it will ignore the `ifOlderThan` value if it is set and the query will be run even if it exists in the cache.

### usePrefetch Example

~~~
function User() {
  const prefetchUser = usePrefetch("getUser");

  // Low priority hover will not fire unless the last request happened more than 35s ago
  // High priority hover will _always_ fire
  return (
    <div>
      <button onMouseEnter={() => prefetchUser(4, { ifOlderThan: 35 })}>
        Low priority
      </button>
      <button onMouseEnter={() => prefetchUser(4, { force: true })}>
        High priority
      </button>
    </div>
  );
}
~~~
