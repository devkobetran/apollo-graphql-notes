---
sidebar_position: 6
---

# Federating the Monolith

## Airlock, the Monolith

[Airlock, the Monolith](https://www.apollographql.com/tutorials/voyage-part2/01-airlock-the-monolith)

#### Practice

Which of these are common problems that developers experience with a monolithic GraphQL schema?

- Merge conflicts caused by modifying the same schema file as multiple other individuals and teams.
- Lack of focus on domain responsibilities caused by the presence of unrelated types and fields.
- Difficulty navigating to the information they need in a constantly growing schema file.

:::info

- A simple schema can potentially evolve into a big, intimidating schema that is difficult to navigate and work with for different teams.
- Federation solves these problems, but the process will be different when starting with an existing application compared to a greenfield app.
- We want to avoid any issues with the client by following a migration plan that swaps the original monolith server with the router.

:::

## Monolith graph setup

[Monolith graph setup](https://www.apollographql.com/tutorials/voyage-part2/02-monolith-graph-setup)

## Monolith as a subgraph

[Monolith as a subgraph](https://www.apollographql.com/tutorials/voyage-part2/03-monolith-as-a-subgraph)

#### Practice

Which of these does the migration plan enable us to do when converting the monolith into a supergraph?

- It preserves all of the schema's current types, fields, and features.
- It ensures that nothing changes about how the client makes requests.

## Auth in a supergraph

[Auth in a supergraph](https://www.apollographql.com/tutorials/voyage-part2/04-auth-in-a-supergraph)

- **Authentication** is determining whether a given user is logged in, and subsequently determining which user someone is.
- **Authorization** is determining what a given user has permission to do or see. (They're allowed to do what they're trying to do.)
- **Field-level authorization**: each resolver checks whether the logged-in user has permission to access that part of the graph.

### Authenticating the user

1. Query sent with Authorization headers
2. Router receives this request and builds a query plan.

### Sending HTTP headers to subgraphs

3. The router's config file is set to propagate Authorization headers.
   - The router sends the Authorization header to its subgraphs with every request.

### Over to the subgraph

4. Subgraph attempts to authenticate the user.
   - If the authentication is successful, then the subgraph puts together an object containing user information and makes it available in its context.
   - If the authentication fails, subgraph returns an Authorization Error to the router.
5. The subgraph resolves the operations using authenticated user info and sends data back to the router.

### Back to the Router

6. The subgraph sends back the requested data to the router, and the router continues with its query plan, eventually combining all those responses into a single JSON object.
   - The router sends the final JSON object back to the client.

#### Practice

Which of the following options can be included as part of a router's `config` file?

- Customizing which headers the router can receive and where to send them
- Configuring CORS for permitted request origins

:::info

- HTTP header called **Authorization** contains the current user's auth token.
- The router can be customized with a configuration file, to pass along HTTP headers to its subgraphs.
- The router passes the user's auth token to the subgraph, which checks whether or not the token is valid for login.
  - If the login is unsuccessful, the subgraph throws an `AuthenticationError` and sends it back to the router.
  - If the login is successful, the subgraph adds the current user's information to its `contextValue` object, which is accessible by its resolvers for field-level authorization.

:::

## Configuring the router

[Configuring the router](https://www.apollographql.com/tutorials/voyage-part2/05-configuring-the-router)

#### Practice

**config.yaml**

```
headers:
  all:
    request:
      - propagate:
          named: "airlock-cookie"
```

In the above **config.yaml** file, what does the `propagate` key tell the router to do?

- Send the 'airlock-cookie' header to all subgraphs.

:::info

- To pass down authorization headers from the router to its subgraphs, we need to set up the router's config file.
  - In the config file, we can set the `headers` property and use the `propagate` property.
- A subgraph can access the authorization (and other request) headers from the router through its `ApolloServer` constructor `context` property.

:::

## Subgraph planning

[Subgraph planning](https://www.apollographql.com/tutorials/voyage-part2/06-subgraph-planning)

### Planning and preparation

- Starting with a plan of your end goal for your supergraph architecture is helpful in understanding where to start first.
- Follow incremental adoption: starting with one subgraph and take small action steps, one at a time.

1. Identify entities

- An **entity** is an object type with fields split between multiple subgraphs.
- Each subgraph is responsible for resolving only the fields it contributes to an entity, and each entity instance is uniquely identifiable with a primary key field.

#### Which of the following types should be marked as an entity in the schema?

- Booking
- Listing
- Host
- Review
- Guest

2. Identify subgraphs

- accounts
- listings
- bookings
- reviews
- payments

3. Decide which entities to migrate first

## Define entities

[Define entities](https://www.apollographql.com/tutorials/voyage-part2/07-define-entities)

Defining entities is an important step in migrating to a federated architecture. It enables everyone to explore the schema and contribute to it.

## A Stub subgraph

[A Stub subgraph](https://www.apollographql.com/tutorials/voyage-part2/08-stub-subgraph)

- **Stub subgraph**: an empty subgraph equipped with the bare minimum needed to get it up and running.
  - It enables us to start with a clean state, set up any CI/CD pipelines with schema checks and approach schema migration incrementally.

#### Practice

Which of the following does `rover dev` not do?

- Creates a new stub subgraph
- Pushes your changes to your supergraph on GraphOS

:::info
`rover dev` helps us with local supergraph development by composing our subgraph schemas locally and spinning up a router.
:::

## Using `@override` to migrate fields

[Using `@override` to migrate fields](https://www.apollographql.com/tutorials/voyage-part2/09-using-override)

### The `@override` directive

- To migrate fields safely from one subgraph to another, we use the `@override` directive.
- We can apply `@override` to fields of an entity and fields of root operation types (such as `Query` and `Mutation`)
  - It tells the router that a particular field is now resolved by the subgraph that applies `@override`, instead of another subgraph where the field is also defined.
- The `@override` directive takes in an argument called `from`, which will be the name of the subgraph that originally defined the field.
- Example in **subgraph-accounts/schema.graphql**:

```ts
type Query {
  user(id: ID!): User @override(from: "monolith")
}
```

:::note
The `monolith` subgraph schema can stay the same, but the router won't call on it anymore when the `user` field is requested.
:::

### Incremental migration with progressive `@override`

- In production environments, we probably want to take it slower, monitoring performance and issues as we make these changes.
- To do this, we can add another argument to the `@override` directive: `label`.
  - This argument takes in a string value starting with `percent` followed by a number in parentheses.
  - The number represents the percentage of traffic for the field that's resolved by this subgraph.
  - The remaining percentage is resolved by the other (`from`) subgraph.
- Example in **subgraph-accounts/schema.graphql**:

```ts
type Query {
  user(id: ID!): User @override(from: "monolith", label: "percent(25)")
}
```

:::note
The `monolith` subgraph schema still stays the same, but now the router will route its requests to the `monolith` subgraph 75% of the time, and the `accounts` subgraph 25% of the time for the `Query.user` field.
:::

#### Practice

Which of the following is true about the `@override` directive?

- It can be applied on field in an object type.
- It takes an argument called `from` that indicates the subgraph the field is overriding.

:::info

- The `@override` directive can be applied to fields of an entity and fields of root operation types to indicate which subgraph should have the responsibility of resolving it.
- The `@override` directive accepts an argument called `from`, which specifies the name of the subgraph that originally defined the field (and which is being overridden).
- With progressive override, we can provide the `@override` directive with an additional argument, `label`. This specifies the percentage of time the router should call upon the subgraph to resolve the field.

:::

## Implementing resolvers

[Implementing resolvers](https://www.apollographql.com/tutorials/voyage-part2/10-implementing-resolvers)

#### Code Challenge

Refer to this schema for the question below:

```ts
interface Book {
  isbn: ID!
  title: String!
  genre: String!
}

type PictureBook implements Book @key(fields: "isbn") {
  isbn: ID!
  title: String!
  genre: String!
  numberOfPictures: Int
  isInColor: Boolean
}

type YoungAdultNovel implements Book {
  isbn: ID!
  title: String!
  genre: String!
  wordCount: Int
  numberOfChapters: Int
}

type LibraryMember @key(fields: "id") {
  id: ID! @external
  faveBook: Book!
}
```

Write a resolver for LibraryMember.faveBook to return a representation of the entity. The underlying object for each book contains a `hasPictures` property, which you can use to determine if it's a `PictureBook` or a `YoungAdultNovel`.

```ts
const resolvers = {
  LibraryMember: {
    faveBook: (book) => {
      const type = book.hasPictures ? "PictureBook" : "YoungAdultNovel";
      return { __typename: type, isbn: book.isbn };
    },
  },
};
```

:::info

- Any subgraph that contributes fields to an entity needs to define a `__resolveReference` resolver function for that entity.
- An entity representation is an object that includes the entity's `__typename` and `@key` fields.
- An interface needs a `__resolveType` resolver.

:::

## Finishing up the subgraph

[Finishing up the subgraph](https://www.apollographql.com/tutorials/voyage-part2/11-finishing-up-the-subgraph)

:::info
The router configuration file takes an option to define a CORS policy, which allows the origins you specify to communicate with your router.
:::
