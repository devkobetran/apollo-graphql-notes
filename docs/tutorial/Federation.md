---
sidebar_position: 5
---

# Federation from Day One

## Intro to Federation

[Intro to Federation](https://www.apollographql.com/tutorials/voyage-part1/01-intro-to-federation)

### Overview

- **Apollo Federation** is an architecture for creating modular graphs.
  - Your graph is built in smaller pieces that all work together.
  - The supergraph improves the developer experience for teams, making it easier to scale your product.

### Life before Apollo Federation

- "supergraph" and "federated graph" mean the same thing: your graph's functionality is divided across smaller, modular graphs.

### The structure of a supergraph

- A supergraph has two key pieces:
  1. One or more subgraphs
  2. A router

### Subgraphs

- In a supergraph, your schema is built in smaller parts.
  - Each part of the schema is owned by a separate subgraph.
  - A **subgraph** is a standalone GraphQL server with its own schema file, resolvers, and data sources.

### The router

- A supergraph architecture also includes the **router**, which sits between clients and the subgraphs.
  - The router is responsible for accepting incoming operations from clients and splitting them into smaller operations that can each be resolved by a single subgraph.
  - The router does this work with the help of the supergraph schema.
  - The supergraph schema is composed of all the fields and types from each subgraph schema.
  - The supergraph schema is a bit like a map, helping the router determine which subgraph can resolve each field in an operation.

### Why use Apollo Federation?

- Core principles of **Apollo Federation**: the separation of concerns.
- By splitting up our schema into subgraphs, backend teams can work on their own subgraphs independently, without impacting developers working on other subgraphs.
  - And since each subgraph is a separate server, teams have the flexibility to choose the language, infrastructure, and policies that work best for them.

:::info
Benefits of Apollo Federation:

- A smoother developer experience among teams, because there are clearer boundaries of responsibility for different parts of the graph.
- Flexibility in subgraph configuration, which means subgraphs can have different numbers of instances, security protocols, or caching strategies.

:::

## Project setup

[Project setup](https://www.apollographql.com/tutorials/voyage-part1/02-project-setup)

## Agreeing on a schema

[Agreeing on a schema](https://www.apollographql.com/tutorials/voyage-part1/03-agreeing-on-a-schema)

#### Practice

:::note

Subgraph Schemas:

- Subgraph schemas help divide a graph's surface area according to different concerns.
- To decide how to split your schema into multiple subgraphs, you can group types and fields related to similar concerns.
- A subgraph schema should contain the types and fields it is responsible for populating.

:::

## Building out the subgraphs

[Building out the subgraphs](https://www.apollographql.com/tutorials/voyage-part1/04-building-out-the-subgraphs)

:::info

- The `buildSubgraphSchema` function takes an object containing `typeDefs` and `resolvers` and returns a federation-ready subgraph schema.
- This schema includes a number of federation directives and types that enable our subgraph to take full advantage of the power of federation.

:::

#### Practice

- To make an `ApolloServer` instance a subgraph, we install a package called `@apollo/subgraph`.
- From that package, we use a function called `buildSubgraphSchema`, which accepts an object containing `typeDefs` and `resolvers`, and returns a federation-ready subgraph schema.
- We add this to the `ApolloServer` configuration object using the `schema` property.

## Managed Federation & the supergraph

[Managed Federation & the supergraph](https://www.apollographql.com/tutorials/voyage-part1/05-managed-federation-and-the-supergraph)

- When the schema registry gets a new or updated version of a subgraph schema, it starts a process called **composition**.
- The schema registry attempts to combine all of the schemas from the registered subgraphs into a single supergraph schema.
- If composition succeeds and there are no validation errors, the schema registry produces a supergraph schema.

#### Managed Federation process

- Every supergraph includes 1 or more subgraphs, each of which has its own schema.
- With managed federation, each of these schemas is published to the Apollo schema registry.
- Whenever a subgraph schema is published, the schema registry triggers a process called composition.
- If successful, this process results in the creation of a supergraph schema, which is then fetched by the supergraph's router via periodic polling.

:::info

- After creating or updating a subgraph schema, developers use the Rover CLI to publish the subgraph schema to the Apollo schema registry.
- The Apollo schema registry composes the subgraph schemas into a supergraph schema, which the router uses to resolve incoming client requests.
- With managed federation, schema updates to the router are managed by GraphOS and happen with zero downtime.

:::

## Publishing the Subgraphs with Rover

[Publishing the Subgraphs with Rover](https://www.apollographql.com/tutorials/voyage-part1/06-publishing-the-subgraphs-with-rover)

#### Supergraph

- The supergraph can be generated automatically by registering subgraphs in GraphOS.
- The supergraph is the result of composing multiple subgraphs together.
- The supergraph is used by a router to resolve incoming client requests.

:::info

- We can use the `rover subgraph publish` command from the Rover CLI to publish our subgraph schemas to the Apollo schema registry.
- Whenever a new subgraph schema is published, GraphOS composes a new supergraph schema with any subgraphs registered to our supergraph.
- The supergraph schema consolidates all the types and fields across our published subgraphs. It also includes extra directives to help the router determine which subgraphs can resolve each field.

:::

## How the Router resolves data

[How the Router resolves data](https://www.apollographql.com/tutorials/voyage-part1/07-how-the-router-resolves-data)

:::info
The router uses the supergraph schema to resolve incoming GraphQL operations from the client
:::

1. Client sends a query
2. Router checks query against supergraph schema
3. Router builds a **query plan**: a list of smaller GraphQL operations to execute on the subgraphs.

- The query plan also specifies the order in which the subgraph operations need to run.
- Query plan execution: each subgraph resolve their respective fields by using their resolvers and data sources to retrieve and populate the requested data.

4. Subgraphs respond with data and the router combines it all into a single response object
5. Router collects subgraphs data to build the complete response and sends it back to client

## Router configuration and Uplink

[Router configuration and Uplink](https://www.apollographql.com/tutorials/voyage-part1/08-router-configuration-and-uplink)

#### What information does the Query Plan Preview in GraphOS Studio include?

- It shows how the router will resolve an operation by requesting data from subgraphs.

:::info

- The GraphOS Router is an executable binary file that can be downloaded and run locally.
- The Query Plan Preview inspects the GraphQL operation in the Explorer and outputs the query plan the router will execute to resolve the operation.

:::

## Connecting data using entities

[Connecting data using entities](https://www.apollographql.com/tutorials/voyage-part1/09-connecting-data-using-entities)

### What's an entity?

- An **entity** is an object type with fields split between multiple subgraphs.
- A subgraph that defines an entity can do one or both of the following:
  1. Reference the entity
  2. Contribute fields to the entity

### Reference the entity

- Referencing an entity means using it as a return type for another field defined in the subgraph.

### Contribute fields to the entity

- Contributing fields to an entity means that one subgraph adds new fields to an entity that are specific to that subgraph's concerns.

### How to create an entity

- To convert an object into an entity in the subgraph schema, need to:

  1. Define a **primary key**, is the field (or fields) of an entity that can uniquely identify an instance of that entity within a subgraph.

  - The router uses primary keys to collect data from across multiple subgraphs and associate it with a single entity instance.
  - In each of our subgraph schemas, we can define a primary key for an entity, by adding the `@key` directive after the type's name.
  - The `@key` directive needs a property called `fields`, which we'll set to the field we want to use as the entity's primary key.

  :::note
  An entity can have more than 1 primary key.
  :::

  ```ts
  type EntityType @key(fields: "id") {
    id: ID!
  }
  ```

  2. Define a **reference resolver**, which is a special resolver function for an entity when each subgraph contributes fields to an entity.

  - The router uses reference resolvers to directly access the entity fields that each subgraph contributes.
  - Every reference resolver has the name: `__resolveReference`, taking only 3 arguments:
    1. `reference`: The entity representation object that's passed in by the router. This tells the subgraph which instance of an entity is being requested.
    2. `context`: The object shared across all resolvers.
    - (this is the same thing like `contextValue` as in other resolvers!)
    3. `info`: Contains information about the operation's execution state, just like in a normal resolver.

#### What's an entity representation?

- An **entity representation** is an object that the router uses to represent a specific instance of an entity.
- A representation always includes the **typename** for that entity and the `@key` field for the specific instance.
  - The `__typename` field: This field exists on all GraphQL types automatically. It always returns the name of its containing type, as a string.
    - e.g. `Location.__typename` returns "Location".
  - The `@key` field: The key-value pair that a subgraph can use to identify the instance of an entity.
- Entity Representation Example:

```ts
{
  "__typename": "Location",
  "id": "loc-2"
}
```

:::tip

- Analogy: You can think of an entity representation as a passport that the router uses to refer to a particular object between subgraphs.
  - The typename field is like a passport's country of origin. It says which entity the object belongs to.
  - And the `@key` field is like a passport's ID number, uniquely identifying this instance of that entity.

:::

#### Practice

Where should an entity's `__resolveReference` function be defined?

- In each subgraph that contributes fields to the entity.

## Defining an entity

[Defining an entity](https://www.apollographql.com/tutorials/voyage-part1/10-defining-an-entity)

- A **stub** serves as a basic representation of a type that includes just enough information to work with that type in the subgraph.

:::info

- To create an entity, we can use the `@key` directive to specify which field(s) can uniquely identify an object of that type.
- When a subgraph can't be used to resolve any non-`@key` fields of an entity, we pass `resolvable: false` to the `@key` directive definition.

:::

## Entities & The Query Plan

[Entities & The Query Plan](https://www.apollographql.com/tutorials/voyage-part1/11-entities-and-the-query-plan)

- The router begins by building a **query plan** that indicates which requests to send to which subgraphs.

#### Practice

Which of the following steps do NOT occur as part of how the router builds and executes its query plan?

- The router makes individual requests to each subgraph and returns separate JSON objects.
- The router makes a separate request to find out the **\_\_typename** for each type included in the query.

:::info

- When the router needs to query for fields from a different subgraph, it also asks for entity representations from the current subgraph it's querying. These representations will be used in the subsequent operation's `\_entities` field, set as the value for the `representations` argument.
- The reference resolver takes each representation and returns the matching data for its requested fields.

:::

## Referencing Entity

[Referencing Entity](https://www.apollographql.com/tutorials/voyage-part1/12-referencing-an-entity)

#### Practice

- When a subgraph references an entity as a return value, it provides a representation of that entity for the router to use. Which of the following are included in that representation?
  - The entity's `__typename`
  - The entity's `@key` fields
- Which of the below is NOT one of the parameters accepted by the `__resolveReference` function?
  - `args`

:::info

- We can reference an entity in one subgraph as the return value for a type's field.
- Any subgraph that contributes fields to an entity needs to define a `__resolveReference` resolver function for that entity. This resolver is called when the router needs to resolve references to that entity made from within other subgraphs.

:::

## Contributing to an Entity

[Contributing to an Entity](https://www.apollographql.com/tutorials/voyage-part1/13-contributing-to-an-entity)

#### Practice

- In a federated graph, we should define our subgraphs based on concerns instead of types.
- Types containing fields that can be resolved across multiple subgraphs are called entities.
- These types always have a key field that enables different subgraphs to associate data with the same object.
- To keep its supergraph schema up to date, our router can poll the Uplink.

#### Code Challenge

You're working on a federated graph that manages book information. The `authors` subgraph defines an `Author` entity with a primary key field `id` of non-nullable type `ID`. You want to use the `Author` entity in the `books` subgraph. Define the `Author` entity below, and add a new field, `books`, which returns a non-nullable list of non-nullable type `Book`.

```
  # Write your code here!
  type Author @key(fields: "id"){
      id: ID!
      books: [Book!]!
  }
```

:::info

- A subgraph that contributes fields to an entity should define the following:
  - The entity, using the `@key` directive and its primary key fields, as well as the new fields the subgraph defines
  - A `__resolveReference` function to know which particular entity instance a subgraph is resolving fields for. This can be taken care of by default by Apollo Server.
- A federated architecture helps organize and illustrate the relationships between types across our graph in a way that an app developer (or multiple teams of developers!) would want to consume the data.
- When both subgraphs use the same primary key to associate data for a type, the router coordinates data from both sources and bundles it up in a single response.

:::

## Putting it all together

[Putting it all together](https://www.apollographql.com/tutorials/voyage-part1/14-putting-it-all-together)

:::info

- Clients request data from a single GraphQL server: the router.
- The router can set CORS rules to specify which websites can talk to it.
- We can set up these rules (and other configurations) through the router's config file.

:::
