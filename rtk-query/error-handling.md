# ?Error Handling

If your query or mutation happens to throw an error when using fetchBaseQuery, it will be returned in the `error` property of the respective hook. The component will re-render when that occurs, and you can show appropriate UI based on the error data if desired.

~~~
function PostsList() {
  const { data, error } = useGetPostsQuery();

  return (
    <div>
      {error.status} {JSON.stringify(error.data)}
    </div>
  );
}
~~~

~~~
function AddPost() {
  const [addPost, { error }] = useAddPostMutation();

  return (
    <div>
      {error.status} {JSON.stringify(error.data)}
    </div>
  );
}
~~~
