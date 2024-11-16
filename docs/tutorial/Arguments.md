---
sidebar_position: 3
---

# Arguments

## Feature Overview

[Feature Overview](https://www.apollographql.com/tutorials/lift-off-part3/01-feature-overview)

## Updating our schema

[Updating our schema](https://www.apollographql.com/tutorials/lift-off-part3/02-updating-our-schema)

**server/src/schema.ts**

```ts
"A Module is a single unit of teaching. Multiple Modules compose a Track"
type Module {
  id: ID!

  "The Module's title"
  title: String!

  "The Module's length in minutes"
  length: Int
}
```

- Why create a separate `Module` type?
  - A track can contain any number of different modules.
  - A module can belong to more than one track.

:::info

```ts
"The track's complete array of Modules";
modules: [Module!]!;
```

- `!` means not null
  - Thus, the array cannot be null
  - The entries in the array cannot be null.

:::

## GraphQL arguments

[GraphQL arguments](https://www.apollographql.com/tutorials/lift-off-part3/03-graphql-arguments)

### Querying for a specific track

- You can add entry points to the `Query` type in the schema.

### How to use GraphQL arguments

- An argument is a value you provide for a particular field in your query.
- The schema defines the arguments that each of your fields accepts.
- Your resolvers can then use a field's provided arguments to help determine how to populate the data for that field.

:::note
Arguments can provide a user-submitted search term, help you retrieve specific objects, filter through a set of objects, or even transform the field's returned value.
:::

- A query that performs a search usually provides the user's search term as an argument.
- To define an argument for a field in our schema, we add parentheses after the field name.
  - **Example**:
    ```ts
    missions(to: String, scheduled: Boolean)
    ```

### Using arguments

```ts
"Fetch a specific track, provided a track's ID"
track(id: ID!): Track
```

## Resolver args parameter

[Resolver args parameter](https://www.apollographql.com/tutorials/lift-off-part3/04-resolver-args-parameter)

### Updating the `RESTDataSource`

**server/src/datasources/track-api.ts**:

```ts
getTrack(trackId: string) {
  return this.get<TrackModel>(`track/${trackId}`);
}
```

### Adding a new resolver

**server/src/resolver.ts**

```ts
// get a single track by ID, for the track page
track: (_, {id}, {dataSources}) => {
  return dataSources.trackAPI.getTrack(id);
},
```

## Resolver Chains

[Resolver Chains](https://www.apollographql.com/tutorials/lift-off-part3/05-resolver-chains)

### Updating the `RESTDataSource`

- Why we extracted author-fetching logic to a different resolver?
  - To keep resolvers more resilient to future changes
  - To prevent unnecessary REST API calls when a query doesn't ask for author data
  - To keep each resolver lightweight and responsible for specific pieces of data

### Resolver Chains

- We can create another resolver function for `Track.author`.
- This resolver is responsible for retrieving author information for a specific track.
- With that, a resolver chain is formed.

#### Resolver parameters

- A resolver function populates the data for a field in your schema.
- The function has 4 parameters:
  1. `parent` contains the returned data of the previous function in the resolver chain.
  2. `args` is an object that contains all the arguments provided to the field.
  3. `contextValue` is used to access data sources such as a database or REST API.
  4. `info` contains informational properties about the operation state.

### Adding a new resolver to the chain

```ts
modules: ({id}, _, {dataSources}) => {
  return dataSources.trackAPI.getTrackModules(id);
},
```

- Destructure the first parameter to retrieve the `id` property from the parent, that's the `id` of the track.
- We don't need the `args` parameter, so that can be an underscore, and then destructure the third `contextValue` parameter for the `dataSources` property.
- Inside, we can return the results of calling our `dataSources.trackAPI.getTrackModules` method, passing in the `id` for the track.

## Query building in Apollo Sandbox

[Query building in Apollo Sandbox](https://www.apollographql.com/tutorials/lift-off-part3/06-query-building-in-apollo-sandbox)

### Variables

```ts
query GetTrack($trackId: ID!) {
 track(id: $trackId) {

 }
}
```

- The `$` symbol indicates a variable in GraphQL.
- The name after the `$` symbol is the name of our variable, which we can use throughout the query.

:::info

- Variables are denoted by the `$` symbol.
- They are used to provide dynamic values for arguments to avoid including hardcoded values in a query.
- Each one's type must match the type specified in the schema.

:::

#### Exercise

Build a query called `GetMission`.
This query uses a variable called `isScheduled` of type nullable `Boolean`.
It retrieves a `mission` using the `scheduled` argument set to the `isScheduled` variable.
It retrieves the mission's `id` and `codename`.

```js
query GetMission($isScheduled: Boolean){
  mission(scheduled: $isScheduled){
    id
    codename
  }
}
```

## Building the track page

[Building the track page](https://www.apollographql.com/tutorials/lift-off-part3/07-building-the-track-page)

**Operation panel:**

```ts
query GetTrack($trackId: ID!) {
  track(id: $trackId) {
    id
    title
    author {
      id
      name
      photo
    }
    thumbnail
    length
    modulesCount
    numberOfViews
    modules {
      id
      title
      length
    }
    description
  }
}
```

- On the client side, the query above will get the value for the `$trackId` variable from the router path or browser URL.

#### Query from the client

We wrap the query string in the `gql` function and then send it to the server with the `useQuery` hook.

:::note

**Example of `gql` function**:

```ts
export const GET_TRACK = gql(`
  // our query goes here
`);
```

:::

## The useQuery hook - with Variables

[The useQuery hook - with Variables](https://www.apollographql.com/tutorials/lift-off-part3/08-the-usequery-hook-with-variables)

`useQuery` Example:

```ts
import { gql } from "./__generated__/index";
import { useQuery } from "@apollo/client";

const GET_SPACECAT = gql(`
    query getSpacecat($spaceCatId: ID!) {
      spacecat(id: $spaceCatId) {
        name
      }
    }
  `);

const spaceCatId = "kitty-1";

const { loading, error, data } = useQuery(GET_SPACECAT, {
  variables: { spaceCatId },
});
```

#### The `useQuery` hook

:::info

- The useQuery hook returns an object with 3 useful properties:
  - `loading` indicates whether the query has completed and results have been returned.
  - `error` is an object that contains any errors that the operation has thrown.
  - `data` contains the results of the query after it has completed.
- To set variables in our query, we declare them in the second parameter of the `useQuery` hook, inside an options object.

:::
