# ?Pagination

RTK Query does not include any built-in pagination behavior. However, RTK Query does make it straightforward to integrate with a standard index-based pagination API. This is the most common form of pagination that you'll need to implement.

src/app/services/posts.ts
~~~
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
interface Post {
  id: number;
  name: string;
}
interface ListResponse<T> {
  page: number;
  per_page: number;
  total: number;
  total_pages: number;
  data: T[];
}

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "/" }),
  endpoints: (builder) => ({
    listPosts: builder.query<ListResponse<Post>, number | void>({
      query: (page = 1) => `posts?page=${page}`,
    }),
  }),
});

export const { useListPostsQuery } = api;
~~~

src/features/posts/PostsManager.tsx
~~~
const PostList = () => {
  const [page, setPage] = useState(1);
  const { data: posts, isLoading, isFetching } = useListPostsQuery(page);

  if (isLoading) {
    return <div>Loading</div>;
  }

  if (!posts?.data) {
    return <div>No posts</div>;
  }

  return (
    <div>
      {posts.data.map(({ id }) => (
        <div key={id}>
          // ...
        </div>
      ))}
      <button onClick={() => setPage(page - 1)} isLoading={isFetching}>
        Previous
      </button>
      <button onClick={() => setPage(page + 1)} isLoading={isFetching}>
        Next
      </button>
    </div>
  );
};
~~~
