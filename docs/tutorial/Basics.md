---
sidebar_position: 1
---

# Basics

- Refer to Environment Setup for setting up Catstronaut Application.

## Feature Data Requirements

- There will be track cards to build out that contain these information:
  - Title
  - Thumbnail image
  - Length (estimated duration)
  - Module count
  - Author name
  - Author picture

### The Graph

- Think of the app data as a collection of objects (such as learning tracks and authors) and relationships between objects (such as each learning track having an author).
- If we think of each object as a node and each relationship as an edge between two nodes, we can think of our entire data model as a graph of nodes and edges. This is called our application's graph.

## Schema Definition Language (SDL)

### The GraphQL Schema

- A **schema** is like a contract between the server and the client.
  - It defines what a GraphQL API can and can't do, and how clients can request or change data.
  - It's an abstraction layer that provides flexibility to consumers while hiding backend implementation details.
- GraphQL's Schema Definition Language (SDL):
  - A schema is a collection of **object types** that contain **fields**.
  - Each field has a type of its own (e.g. **scalar** such as `Int` or `String` or another object type)
  - Declare a type using the `type` keyword, followed by the name in PascalCase.
    - e.g.
    ```SQL
    type SpaceCat {
        # Fields go here
    }
    ```
  - Fields are declared by their name (camelCase), a colon, and then the type.
  - A field can also contain a list, indicated by square brackets.
  - Fields are not seperated by commas
  - Can indicate whether each field value is nullable or non-nullable
    - If a field should never be null, we add an exclamation mark after its type.
    ```SQL
    type SpaceCat {
        "name is not null"
        name: String!
        age: Int
        missions: [Mission]
    }
    ```

#### Descriptions

- It's good practice to document your schema, in the same way that it's helpful to comment your code.
- It makes it easier for your teammates (and future you) to make sense of what's going on.
- The SDL lets you add descriptions to both types and fields by writing strings (in quotation marks) directly above them.
- Triple "double quotes" allow you to add line breaks for clearer formatting of lengthier comments.

```SQL
"""
I'm a block description
with a line break
"""
```

## Building our Schema

- Follow this tutorial: https://www.apollographql.com/tutorials/lift-off-part1/04-building-our-schema

- Navigate to the `server/src/` directory
  - Then, create a new **schema.ts** file
- To get started with our schema, we'll need a couple packages first: `@apollo/server`, `graphql` and `graphql-tag`.
  - The `@apollo/server` package provides a full-fledged, spec-compliant GraphQL server.
  - The `graphql` package provides the core logic for parsing and validating GraphQL queries.
  - The `graphql-tag` package provides the `gql` template literal that we'll use in a moment.

```
import gql from "graphql-tag";
```

:::note
`gql` is a a tagged template literal, used for wrapping GraphQL strings like the schema definition we're about to write.

This converts GraphQL strings into the format that Apollo libraries expect when working with operations and schemas, and it also enables syntax highlighting.

:::

#### type definitions

```ts
export const typeDefs = gql`
  # Schema definitions go here
`;
```

### Defining `Track` type

```SQL
"A track is a group of Modules that teaches about a specific topic"
type Track {
  id: ID!
  title: String!
  author: Author!
  thumbnail: String
  length: Int
  modulesCount: Int
}
```

### Defining `Author` type

```SQL
"Author of a complete Track or a Module"
type Author {
  id: ID!
  name: String!
  photo: String
}
```

### Defining `Query` type

- The fields of this type are entry points into the rest of our schema.
- These are the top-level fields that our client can query for.

```SQL
type Query {
  "Get tracks array for homepage grid"
  tracksForHome: [Track!]!
}
```

**server/src/schema.ts**:

```ts
import gql from "graphql-tag";

export const typeDefs = gql`
  type Query {
    "Get tracks array for homepage grid"
    tracksForHome: [Track!]!
  }

  "A track is a group of Modules that teaches about a specific topic"
  type Track {
    id: ID!
    "The track's title"
    title: String!
    "The track's main author"
    author: Author!
    "The track's main illustration to display in track card or track page detail"
    thumbnail: String
    "The track's approximate length to complete, in minutes"
    length: Int
    "The number of modules this track contains"
    modulesCount: Int
  }

  "Author of a complete Track"
  type Author {
    id: ID!
    "Author's first and last name"
    name: String!
    "Author's profile picture url"
    photo: String
  }
`;
```

## Apollo Server

Link to tutorial: https://www.apollographql.com/tutorials/lift-off-part1/05-apollo-server

### Backend first steps

- 1st goal is to create a GraphQL server that can:
  1. Receive an incoming GraphQL query from our client
  2. Validate that query against our newly created schema
  3. Populate the queried schema fields with mocked data
  4. Return the populated fields as a response
- The Apollo Server library helps us implement this server quickly, painlessly, and in a production-ready way.
- In the `server/src/` folder, open `index.ts`:

  ```ts
  import { ApolloServer } from "@apollo/server";
  import { startStandaloneServer } from "@apollo/server/standalone";
  import { typeDefs } from "./schema";

  async function startApolloServer() {
    const server = new ApolloServer({ typeDefs });
    const { url } = await startStandaloneServer(server);
    console.log(`
    🚀  Server is running!
    📭  Query at ${url}
  `);
  }
  ```

### Mocking data

- You can configure custom mocked responsed for every schema field.
- You can enable default mocked responses for every schema field.

```ts
import { addMocksToSchema } from "@graphql-tools/mock";
import { makeExecutableSchema } from "@graphql-tools/schema";

const server = new ApolloServer({
  schema: addMocksToSchema({
    schema: makeExecutableSchema({ typeDefs }),
    mocks,
  }),
});

const mocks = {
  Track: () => ({
    id: () => "track_01",
    title: () => "Astro Kitty, Space Explorer",
    author: () => {
      return {
        name: "Grumpy Cat",
        photo:
          "https://res.cloudinary.com/dety84pbu/image/upload/v1606816219/kitty-veyron-sm_mctf3c.jpg",
      };
    },
    thumbnail: () =>
      "https://res.cloudinary.com/dety84pbu/image/upload/v1598465568/nebula_cat_djkt9r.jpg",
    length: () => 1210,
    modulesCount: () => 6,
  }),
};
```

## Apollo Explorer

[Apollo Explorer Tutorial](https://www.apollographql.com/tutorials/lift-off-part1/06-apollo-explorer)

**Benefits of using the GraphOS Studio Explorer**:

- You can build and iterate on queries faster.
- You can step through your schema to discover available types and fields.

## The frontend app

[The Frontend app Tutorial](https://www.apollographql.com/tutorials/lift-off-part1/07-the-frontend-app)

## Apollo Client Setup

[Apollo Client Setup](https://www.apollographql.com/tutorials/lift-off-part1/08-apollo-client-setup)

### The `ApolloClient` class

- Every instance of `ApolloClient` uses an in-memory cache.
  - This enables it to store and reuse query results so it doesn't have to make as many network requests.
  - This makes our app's user experience feel much snappier.
  ```ts
  const client = new ApolloClient({
    uri: "http://localhost:4000",
    cache: new InMemoryCache(),
  });
  ```

### The `ApolloProvider` component

- The `ApolloProvider` component uses React's Context API to make a configured Apollo Client instance available throughout a React component tree.
- Wrap our React components tree in the `ApolloProvider` component to make Apollo Client available to our app's Reacts components.

```ts
root.render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <GlobalStyles />
      <Pages />
    </ApolloProvider>
  </React.StrictMode>
);
```

## Codegen

[Codegen Tutorial](https://www.apollographql.com/tutorials/lift-off-part1/09-codegen)

- Look to GraphQL API's schema as the "single source of truth" for all of the types we could possibly query on the frontend.
  - To achieve this and to keep our frontend's type definitions consistent with the backend, is to use a GraphQL Code Generator.
- `@graphql-codegen/cli` is one such tool that can read in a GraphQL schema, compare it against the queries we're asking our frontend code to run, and generate all of the types that we'll need to use on the frontend.

### Process

Check the tutorial.

## Defining a Query

[Defining a Query Tutorial](https://www.apollographql.com/tutorials/lift-off-part1/09-defining-a-query)

- Best Practices when creating client queries:
  - Test out queries in the GraphOS Studio Explorer and copy them over.
  - Assign each query string to a constant with an ALL_CAPS name.
  - Include only the fields that the client requires.
  - Wrap each query in the `gql` function.

## The useQuery Hook

[The useQuery hook tutorial](https://www.apollographql.com/tutorials/lift-off-part1/10-the-usequery-hook)

- A `useQuery` hook is used to execute queries in the frontend app.
