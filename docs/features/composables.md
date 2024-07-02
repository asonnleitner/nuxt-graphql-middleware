# Composables

nuxt-graphql-middleware provides three fully typed composables to perform
queries and mutations.

## useGraphqlQuery

Executes a query using $fetch and returns the response.

```typescript
const { data } = await useGraphqlQuery('films')
```

Variables can be passed as the second argument:

```typescript
const { data } = await useGraphqlQuery('filmById', { id: '123' })
```

Arguments are properly type checked:

```typescript
// ✅ Everyting correct.
const { data } = await useGraphqlQuery('filmById', { id: '123' })

// ❌ Wrong variable type.
const { data } = await useGraphqlQuery('filmById', { id: 123 })

// ❌ Missing variables.
const { data } = await useGraphqlQuery('filmById')

// ❌ Wrong query name.
const { data } = await useGraphqlQuery('getFilmById', { id: '123' })
```

The return value is also properly typed based on the query response:

```typescript
const { data } = await useGraphqlQuery('filmById', { id: '123' })

// ❌ Property does not exist.
console.log(data.films)

// ❌ Object is possibly null.
console.log(data.allFilms.films)

// ✅ Everyting correct.
console.log(data.allFilms?.films)
```

### Fetch Options

You can also pass an object instead, which allows you to additionally provide
fetch options for the request:

```typescript
const { data } = await useGraphqlQuery({
  name: 'filmById',
  variables: { id: '123' },
  fetchOptions: {
    headers: {
      authorization: 'foobar',
    },
  },
})
```

## useAsyncGraphqlQuery

This is a convenience wrapper for using `useGraphqlQuery` inside `useAsyncData`:

```typescript
const { data } = await useAsyncGraphqlQuery('users')
```

This is identical to:

```typescript
const { data } = await useAsyncData(() => useGraphqlQuery('users'))
```

It also works with variables which may also be reactive, in addition to
providing options for `useAsyncData`:

```typescript
const route = useRoute()

const variables = computed<UserByIdQueryVariables>(() => {
  const id = route.params.id.toString()
  return {
    id,
  }
})

const { data: user } = await useAsyncGraphqlQuery('userById', variables, {
  transform: function (v) {
    return v.data.userById
  },
})
```

This will automatically add the provided variables to the `watch` property in
the `useAsyncData` options. Meaning, the query is automatically refreshed when
`variables` changes.

## useGraphqlMutation

Same usage like useGraphqlQuery:

```typescript
const { data } = await useGraphqlMutation('trackVisit')
```

```typescript
const { data } = await useGraphqlMutation('addToCart', { id: '456' })
```

## useGraphqlState

This composable allows you to set fetch options for the useGraphqlQuery and
useGraphqlMutation composables. One common use case is to pass custom request
headers to the GraphQL middleware request:

```typescript
// plugins/graphqlConfig.ts

export default defineNuxtPlugin((NuxtApp) => {
  // Get the configuration state.
  const state = useGraphqlState()

  if (!state) {
    return
  }

  state.fetchOptions = {
    headers: {
      CustomHeader: 'foobar',
    },
  }
}
```

You can find more examples in the
[composables configuration section](/configuration/composable).
