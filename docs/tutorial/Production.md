---
sidebar_position: 7
---

# Federation in Production

## Airlock in production

[Airlock in production](https://www.apollographql.com/tutorials/voyage-part3/01-airlock-in-production)

#### Practice

- **Reviewing the managed federation process**:
  - A supergraph consists of a router and one or more subgraphs.
  - With managed federation, each subgraph schema is published to the schema registry.
  - This triggers a process called composition, which produces a supergraph schema provided to Apollo Uplink.
  - The router fetches this document from Uplink and starts using it to respond to client requests.
- **Environments**:
  - Airlock uses three distinct environments.
  - Code changes start in a **local development environment**, on a developer's own computer.
  - The **staging environment** is used to test and validate changes before these changes are deployed to the **production** environment, where users can interact with the app in the real world.

## Variants in a supergraph

[Variants in a supergraph](https://www.apollographql.com/tutorials/voyage-part3/02-variants-in-a-supergraph)

### What are graph variants?

- A **graph variant** is an instance of a graph that runs in a specific environment (such as staging or production)
  - In GraphOS, each variant of a graph has its own schema, metrics, change history, and operation history.

### Variants in the supergraph

- We have the option to choose which variant to publish a subgraph schema to.
- If we don't specify a variant name, our changes are published by default to the `current` variant.

#### Practice

- Which of the following statements are true about graph variants in GraphOS?
  - Both federated and non-federated graphs can have graph variants
  - Each graph variant has its own schema, metrics, and change history
  - A graph variant represents an instance of a graph running in a specific environment.

:::info

- A graph variant represents an instance of a graph that runs in a specific environment.
- The managed federation process stays the same as before, with the added option of choosing which variant to publish a schema to.
- Graph variants are available for both federated and non-federated graphs.

:::

## Publishing to a graph variant

[Publishing to a graph variant](https://www.apollographql.com/tutorials/voyage-part3/03-publishing-to-a-graph-variant)

#### Practice

- Which of these is the correct way to publish a subgraph schema to a specific variant?
  - By using the `rover subgraph publish` command and specifying the variant name after the `@` symbol in the graph ref.

## Schema checks in a supergraph

[Schema checks in a supergraph](https://www.apollographql.com/tutorials/voyage-part3/04-schema-checks-in-a-supergraph)

### What are schema checks?

- **Schema checks** are a set of predefined tests that help identify potential failures caused by schema updates.
  - They check for issues like incompatibilities between subgraph schemas or breaking existing client operations.
  - With schema checks, we can ensure that our schema changes won't cause issues when we deploy to production.
- There are three types of schema checks:
  1. **Build checks**: validate that a subgraph's schema changes can still compose successfully with other subgraph schemas in the supergraph.
  - For example, if a new type is added to one subgraph, a build check determines whether that addition is compatible with the rest of the subgraphs in the supergraph. If it isn't, we need to investigate the error and fix it.
  2. **Operation checks**: validate that a schema's changes won't break any operations that existing clients send to the graph.
  - For example, let's say a web client regularly sends a GraphQL query to retrieve data for its homepage. If a schema change involves adding a required argument to a field in that query, it might break that client's existing operation if it doesn't include that argument! An operation check helps us guard against this potential failure, listing out the affected operations and allowing the team to address them.
  3. **Linter checks**: analyze your proposed schema changes for violations of formatting rules and other GraphQL best practices.
  - Some common schema conventions include: writing field names in `camelCase`, type names in `PascalCase`, and enums in `SCREAMING_SNAKE_CASE`.

### Launch process

- A launch represents the complete process of making schema updates to any variant of a graph.
- A launch starts with GraphOS attempting to compose the supergraph.
  - If it fails, we'll see an error and the process stops there.
  - Otherwise, a supergraph schema is produced. The supergraph schema is provided to Uplink and the GraphOS Router will fetch the latest schema.

#### Practice

- **Types of schema checks**:
  - Schema checks identify potential failures from proposed schema changes.
  - Linter checks validate that a schema follows GraphQL conventions.
  - Operation checks validate that existing client operations won't break.
  - Build checks validate that the supergraph schema will still compose successfully.
- Which of the following statements about launches in GraphOS are true?
  - A launch starts with GraphOS attempting to compose the supergraph
  - A launch represents the complete process of making schema updates to any variant of a graph.

:::info

- Schema checks help identify potential failures caused by schema updates before they can cause issues in production.
- Build checks validate that a subgraph's schema changes can still compose successfully with other subgraph schemas.
- Operation checks validate that a schema's changes won't break any operations that existing clients are sending to the graph.
- Linter checks validate that a schema follows formatting rules and conventions.
- To run a schema check, we use the `rover subgraph check` command.
- We can inspect the results of a schema check through the terminal or in the Studio Checks page.
- A launch represents the complete process of making schema updates to any variant of a graph. It's triggered by a subgraph being published to the schema registry.

:::

## Local schema checks

[Local schema checks](https://www.apollographql.com/tutorials/voyage-part3/05-local-schema-checks)

- **Value types** are types that are reused and shared across multiple subgraphs.
  - These types can be object types, enums, unions, or interfaces.

#### Practice

- Which of the following information does the `rover subgraph check` command need?
  - The graph ref of the variant to check against
  - The name of the subgraph
  - The file path to the subgraph schema
- Which types of schema checks does the r`over subgraph check` command run on a non-federated graph?
  - Operation checks

:::info
The `@shareable` directive enables multiple subgraphs to resolve a particular object field (or set of object fields).
:::

## Schema checks in CI

[Schema checks in CI](https://www.apollographql.com/tutorials/voyage-part3/06-schema-checks-in-ci)

#### Practice

- Which of the following are benefits of adding schema checks to the CI pipeline?
  - Schema checks can be automated, reducing human error when forgetting to run checks locally.
  - Schema checks catch potential breaking changes before they make it into your codebase.

:::info

- Schema checks are an essential part of a robust CI/CD pipeline.
- Adding schema checks to your pipeline lets you automatically run them every time someone wants to make changes to your codebase, which helps you gain confidence that new changes won't break things.
- We can use the same `rover subgraph check` command that we used locally in our CI automation to perform schema checks.

:::

## Publishing & Deployment

[Publishing & Deployment](https://www.apollographql.com/tutorials/voyage-part3/07-publishing-and-deployment)

#### Practice

- Which of the following events triggers a launch in GraphOS?
  - When a subgraph schema is published to the registry

:::info

- We can automate publishing a subgraph to the registry inside our CD pipeline using the `rover subgraph publish` command.
- Publishing a subgraph schema to any variant in the registry triggers a launch. We can view the launch sequence in the Launches page of Studio.

:::

## Build check errors

[Build check errors](https://www.apollographql.com/tutorials/voyage-part3/08-composition-check-errors)

### The `@inaccessible` directive

- The `@inaccessible` directive is applied to a field in a subgraph schema.
  - Whenever it's present on a field, composition omits that field from the router's API schema.
  - This means that clients can't include the field in operations.
  - This helps us incrementally add a field to multiple subgraphs without breaking composition.

#### Practice

- Which of the following statements about build checks are true?
  - Build checks verify whether changes made to a subgraph will successfully compose with other subgraphs.
  - Build checks can be integrated into CI/CD workflow.

:::info

- We use the `rover subgraph check` command to perform schema checks locally.
- To add a new shared field to a value type, we should first apply the `@inaccessible` directive to the field.
  - Then, we can incrementally add the new field to each subgraph that defines the value type.
  - Finally, we can remove the `@inaccessible` directive and the field will be officially part of the supergraph schema.

:::

## Operation check errors

[Operation check errors](https://www.apollographql.com/tutorials/voyage-part3/09-operation-check-errors)

- **Operation checks** compare schema changes against client operations made in the past, to ensure that we're not introducing any breaking changes for clients consuming data from our graph.
  - Operation checks assess whether the proposed changes to the subgraph will affect any GraphQL operations that have been executed against the schema within a certain timeframe.

#### Practice

- In which of the following scenarios might you want to override an operation check failure?
  - You know the flagged operation is safe because downstream clients have accounted for the change
  - The flagged operation is used only by clients you no longer actively support.

:::info

- Operation checks consider historical operations against the schema to ensure updates to a subgraph do not introduce breaking changes for clients.
- Errors that occur during an operation check can be overridden in Studio.
- Operation check overrides can be set on an operation-by-operation basis, enabling you to mark specific changes as approved or ignore an operation entirely.

:::

## Observability with GraphOS Studio

[Observability with GraphOS Studio](https://www.apollographql.com/tutorials/voyage-part3/10-observability-with-apollo-studio)

### Field usage

- Field usage metrics fall under two categories:
  - **Field requests** represent how many operations were sent by clients over a given time period that included the field.
  - **Field executions** represent how many times your servers have executed the resolver for the field over a given time period.

#### Practice

- If you needed to know how many times a field has been resolved, which metric would you use?
  - Field executions
- If you needed to know how many operations have included a field, which metric would you use?
  - Field requests
- Which of the following operation metrics does GraphOS provide?
  - Error Percentages
  - Request rates
  - Request latencies
  - The top slowest operations

:::info

- Operation metrics provide an overview of operation request rates, service time, and error percentages within a given time period or for a specific operation.
- Field metrics include field requests (how many operations were sent that included that field) and field executions (how many times the resolver for the field has been executed).

:::
