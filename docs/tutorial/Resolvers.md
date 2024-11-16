---
sidebar_position: 2
---

# Resolvers

## Journey of a GraphQL query

[Journey of a GraphQL Query](https://www.apollographql.com/tutorials/lift-off-part2/01-journey-of-a-graphql-query)

### Client-side

- A client sends queries to the GraphQL Server by:
  - shaping the query as a string that defines the selection set of fields it needs.
  - Then, it sends that query to the server in an HTTP `POST` or `GET` request.

### Server-side

- When our server receives the HTTP request, it extracts the string with the GraphQL query.
  - It parses and transforms it into an AST.
  - An **AST (Abstract Syntax Tree)** is a tree-structured document.
  - With this AST, the server validates the query against the types and fields in our schema.

:::warning
If anything is off (e.g. a requested field is not defined in the schema or the query is malformed), the server throws an error and sends it right back to the app.
:::

- A resolver function's mission is to "resolve" its field by populating it with the correct data from the correct source, such as a database or a REST API.
- When a query executes successfully, a `data` key containing a result object with the same shape as the query is returned by the GraphQL server.

### Back to Client-side

- Our client receives the response with exactly the data it needs and passes that data to the right components to render them.

## Exploring our Data

[Exploring our Data tutorial](https://www.apollographql.com/tutorials/lift-off-part2/02-exploring-our-data)

- The data that resolvers retrieve can come from **data sources**:
  - a database,
  - a third-party API,
  - webhooks,
  - etc.
- A single GraphQL API can connect to multiple data sources.

### Where is our data stored?

- https://odyssey-lift-off-rest-api.herokuapp.com/?_gl=1*vcarxk*_ga*MTg0Mjk4NTE0NS4xNzI5MTMwMzEx*_ga_0BGG5V2W2K*MTcyOTUyNTI2Ny4xMy4wLjE3Mjk1MjUyNjcuMC4wLjA

## Apollo RESTDataSource

- How to access the data from our resolvers?
  - GraphQL Server needs to access the REST API.
  - Could call API directly using `fetch`.
  - Use a helper class called `DataSource`.

:::warning

**N + 1 Problem**

- "1" refers to the call to fetch the top-level field
- "N" is the number of subsequent calls to fetch the subfield for each track.
- This problem is inefficient because making N calls to the exact same endpoint retrieves the exact same data.
  - This could be done with 2 calls.

:::

- If the page doesn't change often, could use a cache to avoid unnecessary calls to the REST API.
  - Could design the REST API to set cache headers for its endpoints.

### Challenges with caching in GraphQL

- **Challenge**: With GraphQL, one query is often composed of a mix of different fields and types, coming from different endpoints, with different cache policies.

### Solution with caching in GraphQL

- Instead of being limited by a `fetch` approach, Apollo provides a GraphQL dedicated `DataSource` class called `RESTDataSource` that will efficiently handle resource caching and deduplication for the REST API calls.

### Benefits of a Resource cache

- A resource cache is useful for a data source because:
  - It helps manage the mix of different endpoints with different cache policies.
  - It prevents unnecessary REST API calls for data that doesn't get updated frequently.
  - It helps resolve query fields that have already been fetched much faster.

## Implementing Our RESTDataSource

[Implementing our RESTDataSource Tutorial](https://www.apollographql.com/tutorials/lift-off-part2/04-implementing-our-restdatasource)

e.g.

```ts
export class TrackAPI extends RESTDataSource {
  baseURL = "https://odyssey-lift-off-rest-api.herokuapp.com/";
}
```

:::note
Be sure that your `TrackAPI` class' `baseURL` value ends with a `/`. This will allow our helper class to make requests and append new paths to the `baseURL` without any errors.
:::

**server/src/datasources/track-api.ts**

```ts
import { RESTDataSource } from "@apollo/datasource-rest";

export class TrackAPI extends RESTDataSource {
  baseURL = "https://odyssey-lift-off-rest-api.herokuapp.com/";

  getTracksForHome() {
    return this.get("tracks");
  }

  getAuthor(authorId: string) {
    return this.get(`author/${authorId}`);
  }
}
```

## The shape of a resolver

[The shape of a resolver](https://www.apollographql.com/tutorials/lift-off-part2/05-the-shape-of-a-resolver)

- A resolver's mission is to populate the data for a field in your schema. Your mission is to implement those resolvers!
- A resolver is a function.
  - It is responsible for populating the data for a single field in your schema.
  - It has the same name as the field that it populates data for.
  - It can fetch data from any data source, then transforms that data into the shape your client requires.

```
Query: {
  // returns an array of Tracks that will be used to populate
  // the homepage grid of our web client
  tracksForHome: () => {},
}
```

### How do resolvers interact with the data source

- Resolver functions have a specific signature with four optional parameters: parent, args, contextValue, and info.
  - `parent` is the returned value of the resolver for this field's parent. This will be useful when dealing with resolver chains.
  - `args` is an object that contains all GraphQL arguments that were provided for the field by the GraphQL operation. When querying for a specific item (such as a specific track instead of all tracks), in client-land we'll make a query with an `id` argument that will be accessible via this `args` parameter in server-land.
  - `contextValue` is an object shared across all resolvers that are executing for a particular operation. The resolver needs this argument to share state, like authentication information, a database connection, or in our case the RESTDataSource.
  - `info` contains information about the operation's execution state, including the field name, the path to the field from the root, and more. It's not used as frequently as the others, but it can be useful for more advanced actions like setting cache policies at the resolver level.

## Implementing query resolvers

[Implementing query resolvers](https://www.apollographql.com/tutorials/lift-off-part2/06-implementing-query-resolvers)

**Destructuring Example**:

```
export const resolvers = {
  Query: {
    // get all tracks, will be used to populate the homepage grid of our web client
    tracksForHome: (_, __, { dataSources }) => {
      return dataSources.trackAPI.getTracksForHome();
    },
  },
};
```

:::note
Our tracksForHome resolver will return the results from that TrackAPI method.
:::

**`Parent` in argument Example**:
**server/src/resolvers.ts**:

```ts
export const resolvers = {
  Query: {
    // get all tracks, will be used to populate the homepage grid of our web client
    tracksForHome: (_, __, { dataSources }) => {
      return dataSources.trackAPI.getTracksForHome();
    },
  },
  Track: {
    author: ({ authorId }, _, { dataSources }) => {
      return dataSources.trackAPI.getAuthor(authorId);
    },
  },
};
```

## Connecting the dots in server-land

[Connecting the dots in server-land](https://www.apollographql.com/tutorials/lift-off-part2/07-connecting-the-dots-in-server-land)

- A `dataSources` object needs to be returned from the server's `context` function in order to gain access to our data sources from each resolver's `contextValue` parameter.

## Querying live data

[Querying live data](https://www.apollographql.com/tutorials/lift-off-part2/08-querying-live-data)

- Query using `RESTDataSource` vs Query using `fetch`
  - Due to caching in the `RESTDataSource` approach, results are returned faster after the first query.
  - Different resolvers can use different methods to access the same REST API.
  - You will get the same response data with both approaches.

## Codegen on the server

[Codegen on the server](https://www.apollographql.com/tutorials/lift-off-part2/08-server-codegen)

## Errors! When something goes wrong

[Errors! When something goes wrong](https://www.apollographql.com/tutorials/lift-off-part2/09-errors-when-queries-go-sideways)

- Within a server response, the `data` key or the `errors` key can be expected in the JSON object.
- `GRAPHQL_VALIDATION_FAILED` error code means the GraphQL operation is not valid against the server's schema. Either a field isn't defined, or you might have a typo somewhere in your query!
