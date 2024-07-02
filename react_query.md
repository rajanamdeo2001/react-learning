### Introduction to React Query

React Query, also known as TanStack Query, is a data-fetching library for React that simplifies data synchronization between your client and server. It provides powerful tools to manage server state in your React applications, handling caching, background updates, and data synchronization effortlessly.

### Basics of React Query

#### Installation

To get started, you need to install the React Query library and its dependencies:

```bash
npm install @tanstack/react-query
```

#### Setting Up the Query Client

React Query requires a Query Client, which is the central piece managing all queries and mutations. You need to wrap your application in a `QueryClientProvider` to provide this client to your React components.

```javascript
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourComponent />
    </QueryClientProvider>
  );
}

export default App;
```

#### Basic Usage: Fetching Data

You can use the `useQuery` hook to fetch data. This hook automatically manages loading, error, and data states.

```javascript
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

function fetchTodos() {
  return axios.get('/api/todos');
}

function Todos() {
  const { isLoading, error, data } = useQuery(['todos'], fetchTodos);

  if (isLoading) return 'Loading...';
  if (error) return 'An error occurred';

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

### Intermediate Concepts

#### Query Keys

Query keys are unique identifiers for your queries. They can be a string or an array of strings and variables. This uniqueness is crucial for caching and background refetching.

```javascript
useQuery(['todos', userId], fetchTodos);
```

#### Query Invalidation

Invalidate queries to refetch data when underlying data might have changed.

```javascript
import { useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();
queryClient.invalidateQueries(['todos']);
```

#### Background Refetching

React Query can refetch data in the background to keep your data fresh.

```javascript
useQuery(['todos'], fetchTodos, {
  refetchInterval: 10000, // Refetch every 10 seconds
});
```

#### Mutations

Mutations are used for creating, updating, or deleting data.

```javascript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import axios from 'axios';

function addTodo(todo) {
  return axios.post('/api/todos', todo);
}

function AddTodo() {
  const queryClient = useQueryClient();
  const mutation = useMutation(addTodo, {
    onSuccess: () => {
      queryClient.invalidateQueries(['todos']);
    },
  });

  const handleSubmit = async (event) => {
    event.preventDefault();
    const todo = { title: event.target.elements.title.value };
    mutation.mutate(todo);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" />
      <button type="submit">Add Todo</button>
    </form>
  );
}
```

### Advanced Topics

#### Query Caching

React Query provides a powerful caching mechanism to store query results.

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      cacheTime: 1000 * 60 * 5, // Cache for 5 minutes
    },
  },
});
```

#### Pagination and Infinite Queries

For paginated or infinite scrolling data, use `useInfiniteQuery`.

```javascript
import { useInfiniteQuery } from '@tanstack/react-query';
import axios from 'axios';

function fetchTodos({ pageParam = 1 }) {
  return axios.get(`/api/todos?page=${pageParam}`);
}

function Todos() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery(['todos'], fetchTodos, {
    getNextPageParam: (lastPage, pages) => lastPage.nextPage ?? false,
  });

  return (
    <>
      {data.pages.map(page => (
        <ul key={page.id}>
          {page.todos.map(todo => (
            <li key={todo.id}>{todo.title}</li>
          ))}
        </ul>
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading more...' : 'Load More'}
      </button>
    </>
  );
}
```

#### Query Filters

React Query supports various query filters to manipulate query data.

```javascript
queryClient.getQueryData(['todos']);
queryClient.removeQueries(['todos']);
queryClient.setQueryData(['todos'], newData);
```

#### React Query Devtools

React Query Devtools provide a visual interface for managing and debugging your queries.

```bash
npm install @tanstack/react-query-devtools
```

```javascript
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  <YourComponent />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

### Advanced Concepts

#### React Query Configuration

Customizing React Query with a configuration object allows you to set default options for queries and mutations.

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 10, // 10 minutes
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 3, // Retry failed mutations 3 times
    },
  },
});
```

#### Optimistic Updates

Optimistic updates improve user experience by immediately updating the UI, assuming the server will succeed.

```javascript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import axios from 'axios';

function updateTodo(id, updates) {
  return axios.patch(`/api/todos/${id}`, updates);
}

function TodoItem({ todo }) {
  const queryClient = useQueryClient();
  const mutation = useMutation(updateTodo, {
    onMutate: async (updatedTodo) => {
      await queryClient.cancelQueries(['todos']);
      const previousTodos = queryClient.getQueryData(['todos']);
      queryClient.setQueryData(['todos'], (old) =>
        old.map((todo) =>
          todo.id === updatedTodo.id ? { ...todo, ...updatedTodo } : todo
        )
      );
      return { previousTodos };
    },
    onError: (err, updatedTodo, context) => {
      queryClient.setQueryData(['todos'], context.previousTodos);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['todos']);
    },
  });

  return (
    <div>
      <span>{todo.title}</span>
      <button
        onClick={() => mutation.mutate({ id: todo.id, title: 'Updated Title' })}
      >
        Update
      </button>
    </div>
  );
}
```

#### Hydration

Hydrating initial data from the server is crucial for SSR (Server-Side Rendering) or SSG (Static Site Generation) with frameworks like Next.js.

```javascript
import { Hydrate, QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function MyApp({ Component, pageProps }) {
  return (
    <QueryClientProvider client={queryClient}>
      <Hydrate state={pageProps.dehydratedState}>
        <Component {...pageProps} />
      </Hydrate>
    </QueryClientProvider>
  );
}

export default MyApp;
```

### Advanced Query Management

#### Suspense Mode

React Query integrates with React's Suspense for data fetching, providing a seamless loading experience.

```javascript
import { Suspense } from 'react';
import { useQuery } from '@tanstack/react-query';

function Todos() {
  const { data } = useQuery(['todos'], fetchTodos, { suspense: true });

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Todos />
    </Suspense>
  );
}
```

#### Query Cancellation

React Query allows query cancellation to avoid unnecessary network requests.

```javascript
function fetchTodos(signal) {
  return axios.get('/api/todos', { signal });
}

function Todos() {
  const { data, isFetching, refetch } = useQuery(['todos'], fetchTodos);

  return (
    <div>
      {isFetching ? 'Loading...' : data.map(todo => <li key={todo.id}>{todo.title}</li>)}
      <button onClick={() => refetch()}>Refetch</button>
    </div>
  );
}
```

### Best Practices

1. **Use Descriptive Query Keys**: Always use descriptive and unique query keys to avoid conflicts.
2. **Batching Requests**: Batch multiple requests to reduce network overhead.
3. **Error Handling**: Implement robust error handling strategies using React Query's `onError` and `onSettled` callbacks.
4. **Query Dependencies**: Use dependent queries to fetch data in a specific order.
5. **Use Selectors**: Use the `select` option to transform data before it reaches the component.

### Use Cases and Applications

1. **Dashboard Applications**: React Query is perfect for dashboards that need to fetch and update data frequently.
2. **E-commerce Platforms**: Managing product listings, user carts, and orders with real-time updates.
3. **Social Media Applications**: Fetching posts, comments, and notifications efficiently.
4. **Content Management Systems**: Synchronizing state between client and server for editing and publishing content.

### Conclusion

React Query is a powerful tool that simplifies data fetching and state management in React applications. Its caching, background refetching, and mutation handling capabilities help build robust, scalable applications. By understanding the basics, intermediate concepts, and advanced features, you can effectively manage server state and improve the performance of your React applications.

Would you like more detailed examples or explanations of specific features?
