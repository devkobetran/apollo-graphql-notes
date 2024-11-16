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
