---
sidebar_position: 1
---

# Introduction

## Introduction to Apollo GraphQL

## Catstronauts Example Application

- Catstronauts will be a full stack GraphQL Example Application of focus within this docusaurus notes.

### Schema First Design

- **"Schema-first" design** is implements the feature based on exactly which data our client application needs.

1. **Defining the schema**: We identify which data our feature requires, and then we structure our schema to provide that data as intuitively as possible.
2. **Backend implementation**: We build out our GraphQL API using Apollo Server and fetch the required data from whichever data sources contain it. In this first course, we will be using mocked data. In a following course, we'll connect our app to a live REST data source.
3. **Frontend implementation**: Our client consumes data from our GraphQL API to render its views.

### Benefits of schema-first design

- Reduces total development time by allowing frontend and backend teams to work in parallel.
  - The frontend team can start working with mocked data as soon as the schema is defined
  - The backend team develops the API based on that same schema.
