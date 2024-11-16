---
sidebar_position: 4
---

# Mutations

## What is a mutation?

- A **mutation** is a write operation in GraphQL.
- The `Mutation` type serves as an entry point to our schema.

## Schema Syntax

```ts
type Mutation {
    addSpacecat(name: String!): Spacecat
}
```

:::tip
Start the operation name with a varb like add, delete, or create.

Examples: `createMission`, `deleteMission`
:::

## Modifying multiple objects

```ts
type Spacecat{
    id: ID!
    name: String!
    missions:[Mission]
}

type Mission {
    id: ID!
    codename: String!
    crewMembers: [Spacecat]
}
```

- Example: create a mutation called `assignMission` which will return a type of both `Spacecat` and `Mission`
- Include 3 fields to all mutation responses:
  - `code`: an `Int` that refers to the status of the response, similar to an HTTP status code.
  - `success`: a `Boolean` flag that indicates whether all the updates the mutation was responsible for succeeded.
  - `message`: a `String` to display information about the result of the mutation on the client side. This is particularly useful if the mutation was only partially successful and a generic error message can't tell the whole story.

:::info
We need to create a separate object for a mutation's return type because:

- The return type can contain partial errors and partial successful data
- When the mutation modifies multiple objects, it can return all of those objects

:::

- Here is the new type created for the `assignMission` mutation:

```ts
type AssignMissionResponse{
    code: Int!
    Success: Boolean!
    message: String!
    spacecat: Spacecat
    mission: Mission
}
```

#### Mutations vs Queries

:::note

- Queries and mutations are both types of GraphQL operations.
- Queries are read operations that always retrieve data.
- Mutations are write operations that always modify data.
- Similar to `Query` fields, fields of the `Mutation` type are also entry points into a GraphQL API.

:::

## Adding a mutation to our schema

### Updating our schema

- Example of a mutation that needs to know which track to update.

```ts
incrementTrackViews(id: ID!)
```

- Return type for the above mutation response

```ts
type IncrementTrackViewsResponse {
    "Similar to HTTP status code, represents the status of the mutation"
    code: Int!
    "Indicates whether the mutation was successful"
    success: Boolean!
    "Human-readable message for the UI"
    message: String!
    "Newly updated track after a successful mutation"
    track: Track
}
```

:::warning

In the mutation response type (`IncrementTrackViewsResponse`) above, the modified object's return type is nullable (`track: Track`) because the mutation might encounter errors that prevent a `Track` from being modified.

:::

- **server/src/schema.ts**:

```ts
type Mutation {
  incrementTrackViews(id: ID!): IncrementTrackViewsResponse!
}
```

#### Code Challenge

```ts
const typeDefs = gql`
  # write your mutation here
  type Mutation {
    assignSpaceship(spaceshipId: ID!, missionId: ID!): AssignSpaceshipResponse!
  }

  # write your mutation return type here
  type AssignSpaceshipResponse {
    code: Int!
    success: Boolean!
    message: String!
    spaceship: Spaceship
    mission: Mission
  }

  type Mission{ 
    ...
  }

  type Spaceship{
    ...
  }
`;
```

## Updating our TrackAPI data source

[Updating our TrackAPI data source](https://www.apollographql.com/tutorials/lift-off-part4/04-updating-our-trackapi-data-source)

### Updating the data source

- Why is a separate `RESTDataSource` class used to handle data retrieval?
  - To keep data-fetching implementations in a dedicate class and keep resolvers simple and clean.
  - It automatically handles resource caching and request deduplication for our REST API calls.

**server/src/datasources/track-api.ts**:

```ts
incrementTrackViews(trackId: string) {
  return this.patch<TrackModel>(`track/${trackId}/numberOfViews`);
}
```

- We need to make an HTTP `PATCH` request by calling `this.patch` which is provided to us by the `RESTDataSource` class we inherited from.
- Inside the parentheses, we give it the endpoint.

## Resolving a mutation successfully

[Resolving a mutation successfully](https://www.apollographql.com/tutorials/lift-off-part4/05-resolving-a-mutation-successfully)

### Mutation resolvers

- Template:

```ts
const resolvers: Resolvers = {
  Query: {
    // ... query resolvers
  },
  Mutation: {
    // where our new resolver function will go
    // increments a track's numberOfViews property
    incrementTrackViews: (parent, args, contextValue, info) => {},
  },
};
```

:::info
In the structure of the `resolvers` object:

- Resolver function names must match the field name in the schema.
- The `Query` and `Mutation` types in the schema should have corresponding keys in the `resolvers` object.

:::

:::note

```ts
dataSources.trackAPI.incrementTrackViews(id);
```

Why can't this resolver immediately return the results of the `TrackAPI` call in this case?

- It builds part of the response from the REST operation status.
- The schema is also expecting `code`, `success`, `message` fields.

:::

### Fulfilling the schema requirements

#### Spaceship & Mission Example

- Add a resolver for the new `assignSpaceship` mutation provided in the schema, using arrow function syntax.
- Use the `dataSources.spaceAPI` class and its method `assignSpaceshipToMission`, which takes the `spaceshipId` and the `missionId` as arguments (in that order)
- This method returns an object with the newly updated `spaceship` and `mission`.
- Follow the schema requirements to return an object for the successful result.
- `code` should be `200`, success should be `true` and `message` should say `Successfully assigned spaceship ${spaceshipId} to mission ${missionId}`

```ts
import type { Resolvers } from "./__generated__/index";
import { gql } from "graphql-tag";

const typeDefs = gql`
  type Mutation {
    assignSpaceship(spaceshipId: ID!, missionId: ID!): AssignSpaceshipResponse
  }

  type AssignSpaceshipResponse {
    code: Int!
    success: Boolean!
    message: String!
    spaceship: Spaceship
    mission: Mission
  }
`;

const resolvers: Resolvers = {
  Mutation: {
    assignSpaceship: async (_, { spaceshipId, missionId }, { dataSources }) => {
      const { spaceship, mission } =
        await dataSources.spaceAPI.assignSpaceshipToMission(
          spaceshipId,
          missionId
        );
      return {
        code: 200,
        success: true,
        message: `Successfully assigned spaceship ${spaceshipId} to mission ${missionId}`,
        spaceship: spaceship,
        mission: mission,
      };
    },
  },
};
```

## Resolving a mutation with errors

[Resolving a mutation with errors](https://www.apollographql.com/tutorials/lift-off-part4/06-resolving-a-mutation-with-errors)

### Handling the error case

:::info

- Instead of hardcoding the error code as 404, be dynamic by using the value that Apollo Server and the RESTDataSource class provides.
- The `extensions` field to the error contains relevant error details such as a `response` property and the `status` property which outputs the status code.

```ts
code: err.extensions.response.status;
```

- For the `message` property, be dynamic by returning different types of error messages using:

```ts
message: err.extensions.response.body;
```

:::

- `incrementTrackViews` resolver with error handling:

```ts
incrementTrackViews: async (_, {id}, {dataSources}) => {
  try {
    const track = await dataSources.trackAPI.incrementTrackViews(id);
    return {
      code: 200,
      success: true,
      message: `Successfully incremented number of views for track ${id}`,
      track
    };
  } catch (err) {
    return {
      code: err.extensions.response.status,
      success: false,
      message: err.extensions.response.body,
      track: null
    };
  }
},
```

:::note
The `incrementTrackViews` resolver above handles both a successful response and possible errors.

- Both objects fulfill the requirements of the schema.
- Only the successful response contains `track` data.

:::

## Testing a Mutation in the Explorer

[Testing a Mutation in the Explorer](https://www.apollographql.com/tutorials/lift-off-part4/07-testing-a-mutation-in-the-explorer)

### Building a GraphQL mutation

- When writing a GraphQL mutation, use the `mutation` keyword and immediately following after the keyword should be the operation name.

```ts
mutation IncrementTrackViews($incrementTrackViewsId: ID!){
  incrementTrackViews(id: $incrementTrackViewsId)
}
```

- In the successful mutation response, the values of `code`, `success`, and `message` come from the `incrementTrackViews` resolver.
- When the mutation fails, the values of `code` and `message` come from the `error.extensions.response` property in the mutation's resolver.

## The useMutation hook

[The useMutation hook](https://www.apollographql.com/tutorials/lift-off-part4/08-the-usemutation-hook)

### Mutation in client-land

```ts
/**
 * Mutation to increment a track's number of views
 */
const INCREMENT_TRACK_VIEWS = gql(`
  mutation IncrementTrackViews($incrementTrackViewsId: ID!) {
    incrementTrackViews(id: $incrementTrackViewsId) {
      code
      success
      message
      track {
        id
        numberOfViews
      }
    }
  }
`);
```

### The `useMutation` hook

- `useMutation` does not execute the mutation automatically.
- `useMutation` hook returns an array with two elements:
  1. The first element is the **mutate function**.
  2. The second element is an object with information about the mutation: `loading`, `error`, and `data`.

### Setting up the `onClick`

```ts
<CardContainer
  to={`/track/${id}`}
  onClick={() => incrementTrackViews()}
>
```

#### Sending a mutation client-side

:::info

- We use hooks to send requests to our GraphQL API from a React client.
- TO send a mutation, we use the `useMutation` hook.
- This returns an array, where the first element is the mutate function used to trigger the mutation.
- The second element is an object with more information about the mutation, such as `loading`, `error`, and `data`.
- This hook takes in a GraphQL operation as the first parameter.
- It also takes in an `options` object as the second parameter, where properties like `variables` are set.

:::

:::note

#### `useQuery` vs `useMutation` hooks

- The `useQuery` hook is used to send queries, whereas the `useMutation` hook is used to send mutations.
- The `useQuery` hook runs automatically on component render, whereas the `useMutation` hook returns a mutation function needed to trigger the mutation.
- The `useQuery` hook returns an object, whereas the `useMutation` hook returns an array.

:::

### Console loading

```ts
const [incrementTrackViews] = useMutation(INCREMENT_TRACK_VIEWS, {
  variables: { incrementTrackViewsId: id },
  // to observe what the mutation response returns
  onCompleted: (data) => {
    console.log(data);
  },
});
```

#### Example

- Use the useMutation hook to send the `ASSIGN_SPACESHIP_MUTATION` mutation to the server.
- It takes 2 variables:
  - `spaceshipId`
  - `missionId`
- Destructure the mutate function (call it `assignSpaceship`), as well as the `loading`, `error` and `data` properties from the return array of the hook.

```ts
import { useMutation } from "@apollo/client";
import { gql } from "./__generated__/index";

const ASSIGN_SPACESHIP_MUTATION = gql(`
  mutation AssignSpaceshipToMissionMutation($spaceshipId: ID!, $missionId: ID!) {
    assignSpaceship(spaceshipId: $spaceshipId, missionId: $missionId) {
      code
      success
      message
      spaceship {
        name
      }
      mission {
        codename
      }
    }
  }
`);

const spaceshipId = "ROCKET_X";
const missionId = "M0007";

const [assignSpaceship, { data, loading, error }] = useMutation(
  ASSIGN_SPACESHIP_MUTATION,
  { variables: { spaceshipId, missionId } }
);
```

## Our mutation in the browser

[Our mutation in the browser](https://www.apollographql.com/tutorials/lift-off-part4/09-our-mutation-in-the-browser)

### Apollo Client Cache

- How do we see the number of views for a track update while we are on the page?
  - The value is first loaded from cache, then updates when the mutation response comes back successfully.
