# React Redux

## Getting Started

```
npm react-redux @reduxjs@toolkit
```

## Creating a Redux Store

```js
// app/store.js

import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
})
```

```js
// index.js

import React from 'react'
import ReactDOM from 'react-dom'
import './index.css'
import App from './App'
import { store } from './app/store'
import { Provider } from 'react-redux'

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
)
```

## Creating a Redux Slice

```js
// features/couter/counterSlice.js

import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  count: 0,
}

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  reducers: {
    increment: state => {
      state.count += 1
    },
    decrement: state => {
      state.count -= 1
    },
  },
})

export const { increment, decrement } = counterSlice.actions

export default counterSlice.reducer
```

## Dispatching Actions in a Component

```js
import { useSelector, useDispatch } from 'react-redux'
import { increment, decrement } from './counterSlice'

const Counter = () => {
  const count = useSelector(state => state.counter.count)
  const dispatch = useDispatch()

  return (
    <>
      <p>{count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </>
  )
}
```

## Prepare Callback

```js
reducers: {
    postAdded: {
        reducer(state, action) {
            state.push(action.payload)
        },
        prepare(title, content, userId) {
            return {
                payload: {
                    id: nanoid(),
                    title,
                    content
                    }
                }
            }
        },
}

// before
dispatch(postAdded(id: nanoId(), title, content))
// after
dispatch(postAdded(title, content))
```

## Thunk Middleware

```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

const initialState = {
  posts: [],
  status: 'idle', //'idle' | 'loading' | 'succeeded' | 'failed'
  error: null,
}

export const fetchPosts = createAsyncThunk('posts/fetchPosts', async () => {
  const response = await axios.get(POSTS_URL)
  return response.data
})

const postsSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {},
  extraReducers(builder) {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        state.status = 'loading'
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded'
        state.posts = action.payload
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed'
        state.error = action.error.message
      })
  },
})
```

## Performance Techinques & Optimization

### Creating a Memoized Selector

```js
import { createSelector } from '@reduxjs/toolkit'

export const selectPostsByUser = createSelector(
  [selectAllPosts, (state, userId) => userId],
  (posts, userId) => posts.filter(post => post.userId === userId)
)
```

### Normalizations

```js
import { createEntityAdapter } from '@reduxjs/toolkit'

const postsAdapter = createEntityAdapter({
  sortComparer: (a, b) => b.date.localeCompare(a.date),
})

const initialState = postsAdapter.getInitialState({
  status: 'idle', //'idle' | 'loading' | 'succeeded' | 'failed'
  error: null,
  count: 0,
}) // entites already exist

// find
const existingPost = state.entities[postId]

// CRUD
postsAdapter.upsertMany(state, action.payload)
postsAdapter.addOne(state, action.payload)
postsAdapter.upsertOne(state, action.payload)
postsAdapter.removeOne(state, id)

// complete reference
// addOne,
// addMany,
// setOne,
// setMany,
// setAll,
// removeOne,
// removeMany,
// removeAll,
// updateOne,
// updateMany,
// upsertOne,
// upsertMany

// selectors

// getSelectors creates these selectors and we rename them with aliases using destructuring
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds,
  // Pass in a selector that returns the posts slice of state
} = postsAdapter.getSelectors(state => state.posts)
```

## RTK Query

### Create an Api Slice

```js
// features/api/apiSlice

import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: 'http://localhost:3500' }),
  tagTypes: ['Todos'],
  endpoints: builder => ({
    // Read
    getTodos: builder.query({
      query: () => '/todos',
      transformResponse: res => res.sort((a, b) => b.id - a.id),
      providesTags: ['Todos'],
    }),
    // Create
    addTodo: builder.mutation({
      query: todo => ({
        url: '/todos',
        method: 'POST',
        body: todo,
      }),
      invalidatesTags: ['Todos'],
    }),
    // Update
    updateTodo: builder.mutation({
      query: todo => ({
        url: `/todos/${todo.id}`,
        method: 'PATCH',
        body: todo,
      }),
      invalidatesTags: ['Todos'],
    }),
    deleteTodo: builder.mutation({
      // Delete
      query: ({ id }) => ({
        url: `/todos/${id}`,
        method: 'DELETE',
        body: id,
      }),
      invalidatesTags: ['Todos'],
    }),
  }),
})

export const {
  useGetTodosQuery,
  useAddTodoMutation,
  useUpdateTodoMutation,
  useDeleteTodoMutation,
} = apiSlice
```

### Api Provider

```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import './index.css'
import App from './App'

import { ApiProvider } from '@reduxjs/toolkit/query/react'
import { apiSlice } from './features/api/apiSlice'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <ApiProvider api={apiSlice}>
      <App />
    </ApiProvider>
  </React.StrictMode>
)
```

### Use Hooks in Components​

```js
import {
  useGetTodosQuery,
  useUpdateTodoMutation,
  useDeleteTodoMutation,
  useAddTodoMutation,
} from '../api/apiSlice'

// Read
const { data: todos, isLoading, isSuccess, isError, error } = useGetTodosQuery()

// Create
const [addTodo] = useAddTodoMutation()
addTodo({ userId: 1, title: newTodo, completed: false })

// Update
const [updateTodo] = useUpdateTodoMutation()
updateTodo({ ...todo, completed: !todo.completed })

// Delete
const [deleteTodo] = useDeleteTodoMutation()
deleteTodo({ id: todo.id })
```
