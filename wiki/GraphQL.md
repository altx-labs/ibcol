# GraphQL Integration

The IBCOL platform uses GraphQL for data fetching and mutations. This document explains how GraphQL is integrated into the application and how to work with it.

## GraphQL Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Apollo Client  │────▶│  GraphQL API    │────▶│  Database       │
│  (React)        │     │  (Microservice) │     │  (MongoDB)      │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

The GraphQL integration consists of:

1. **Apollo Client**: Client-side GraphQL library
2. **GraphQL API**: External microservice ([ibcol-micro-graphql-api](https://github.com/altx-labs/ibcol-micro-graphql-api))
3. **GraphQL Schema**: Defines available queries and mutations
4. **Apollo Provider**: React context provider for Apollo Client

## Apollo Client Setup

The Apollo Client is set up in `helpers/withApollo.js`:

```javascript
import { ApolloClient } from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { HttpLink } from 'apollo-link-http';
import { setContext } from 'apollo-link-context';
import withApollo from 'next-with-apollo';
import getConfig from 'next/config';

const { publicRuntimeConfig } = getConfig();

export default withApollo(({ initialState, headers }) => {
  const httpLink = new HttpLink({
    uri: publicRuntimeConfig.GRAPHQL_URL,
    credentials: 'same-origin',
  });

  const authLink = setContext((_, { headers }) => {
    // Get the authentication token from local storage if it exists
    const token = localStorage.getItem('token');
    
    // Return the headers to the context so httpLink can read them
    return {
      headers: {
        ...headers,
        authorization: token ? `Bearer ${token}` : "",
      }
    }
  });

  return new ApolloClient({
    link: authLink.concat(httpLink),
    cache: new InMemoryCache().restore(initialState || {}),
  });
});
```

## Apollo Provider

The Apollo Provider is set up in `_app.js`:

```jsx
import { ApolloProvider } from 'react-apollo';
import withApollo from 'helpers/withApollo';

class MyApp extends App {
  render() {
    const { Component, pageProps, apollo } = this.props;
    
    return (
      <Container>
        <ApolloProvider client={apollo}>
          {/* Application components */}
        </ApolloProvider>
      </Container>
    );
  }
}

export default withApollo(MyApp);
```

## GraphQL Queries

Queries are defined using the `gql` tag and executed using the `Query` component:

```jsx
import { Query } from 'react-apollo';
import gql from 'graphql-tag';

const GET_APPLICATIONS = gql`
  query GetApplications($accessToken: TokenInput!) {
    getApplications(accessToken: $accessToken) {
      _id
      teamName
      ref
      createdAt
      updatedAt
    }
  }
`;

// In a component:
<Query 
  query={GET_APPLICATIONS} 
  variables={{ 
    accessToken: { 
      email: user.email, 
      token: user.token 
    } 
  }}
>
  {({ loading, error, data }) => {
    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error.message}</p>;
    
    return (
      <ul>
        {data.getApplications.map(app => (
          <li key={app._id}>{app.teamName}</li>
        ))}
      </ul>
    );
  }}
</Query>
```

## GraphQL Mutations

Mutations are defined using the `gql` tag and executed using the `Mutation` component:

```jsx
import { Mutation } from 'react-apollo';
import gql from 'graphql-tag';

const ADD_APPLICATION = gql`
  mutation AddApplication($application: ApplicationInput!) {
    addApplication(application: $application) {
      teamName
      ref
    }
  }
`;

// In a component:
<Mutation mutation={ADD_APPLICATION}>
  {(addApplication, { loading, error }) => (
    <form
      onSubmit={e => {
        e.preventDefault();
        addApplication({ 
          variables: { 
            application: this.state.formData 
          } 
        });
      }}
    >
      {/* Form fields */}
      <button type="submit" disabled={loading}>
        Submit
      </button>
      {error && <p>Error: {error.message}</p>}
    </form>
  )}
</Mutation>
```

## GraphQL Schema

The GraphQL schema defines the available queries and mutations. Here are some key types:

```graphql
# Application Input
input ApplicationInput {
  teamName: String!
  contactEmail: String!
  contactPhone: String
  country: String!
  studentRecords: [StudentRecordInput!]!
  advisorRecords: [AdvisorRecordInput!]
  projectRecords: [ProjectRecordInput!]!
}

# Student Record Input
input StudentRecordInput {
  firstName: String!
  lastName: String!
  email: String!
  phone: String
  university: String!
  department: String!
  studentCardFrontFileId: String
  studentCardBackFileId: String
  transcriptFileId: String
}

# Project Record Input
input ProjectRecordInput {
  title: String!
  description: String!
  category: String!
  whitepaperFileId: String
}

# Queries
type Query {
  getApplications(accessToken: TokenInput!): [Application!]!
  getApplication(accessToken: TokenInput!, applicationId: ID!): Application
}

# Mutations
type Mutation {
  addApplication(application: ApplicationInput!): Application!
  updateApplication(accessToken: TokenInput!, application: ApplicationUpdateInput!): Application!
  deleteApplication(accessToken: TokenInput!, applicationId: ID!): Boolean!
}
```

## Authentication with GraphQL

Authentication is handled using token-based authentication:

```javascript
// Login mutation
const LOGIN = gql`
  mutation Login($email: String!, $password: String!) {
    login(email: $email, password: $password) {
      token
      user {
        id
        email
        role
      }
    }
  }
`;

// Using the login mutation
<Mutation mutation={LOGIN}>
  {(login, { loading, error }) => (
    <form
      onSubmit={e => {
        e.preventDefault();
        login({ 
          variables: { 
            email: this.state.email,
            password: this.state.password
          } 
        })
        .then(({ data }) => {
          // Store the token
          localStorage.setItem('token', data.login.token);
          // Redirect to dashboard
          Router.pushRoute('adminDashboard', { locale: this.props.locale });
        });
      }}
    >
      {/* Form fields */}
      <button type="submit" disabled={loading}>
        Login
      </button>
      {error && <p>Error: {error.message}</p>}
    </form>
  )}
</Mutation>
```

## Error Handling

GraphQL errors are handled at multiple levels:

1. **Network Errors**: Handled by Apollo Client
2. **GraphQL Errors**: Returned in the response
3. **Component-Level Errors**: Handled in the component

Example of error handling:

```jsx
<Mutation mutation={ADD_APPLICATION}>
  {(addApplication, { loading, error }) => {
    // Handle different types of errors
    let errorMessage = null;
    
    if (error) {
      if (error.networkError) {
        errorMessage = "Network error. Please check your connection.";
      } else if (error.graphQLErrors) {
        errorMessage = error.graphQLErrors.map(err => err.message).join(", ");
      } else {
        errorMessage = error.message;
      }
    }
    
    return (
      <form onSubmit={/* ... */}>
        {/* Form fields */}
        <button type="submit" disabled={loading}>
          Submit
        </button>
        {errorMessage && <p className="error">{errorMessage}</p>}
      </form>
    );
  }}
</Mutation>
```

## Caching

Apollo Client includes a caching system that:

1. Stores query results in memory
2. Automatically updates the UI when data changes
3. Supports manual cache updates for optimistic UI

Example of updating the cache after a mutation:

```jsx
<Mutation
  mutation={ADD_APPLICATION}
  update={(cache, { data: { addApplication } }) => {
    // Read the current cache
    const { getApplications } = cache.readQuery({
      query: GET_APPLICATIONS,
      variables: { accessToken }
    });
    
    // Write back to the cache with the new application
    cache.writeQuery({
      query: GET_APPLICATIONS,
      variables: { accessToken },
      data: { getApplications: getApplications.concat([addApplication]) }
    });
  }}
>
  {/* Mutation component content */}
</Mutation>
```

## Optimistic UI

For better user experience, optimistic UI updates can be implemented:

```jsx
<Mutation
  mutation={ADD_APPLICATION}
  optimisticResponse={{
    addApplication: {
      __typename: 'Application',
      _id: 'temp-id',
      teamName: this.state.formData.teamName,
      ref: 'pending',
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    }
  }}
  update={/* ... */}
>
  {/* Mutation component content */}
</Mutation>
```

## GraphQL Fragments

For reusable pieces of queries, GraphQL fragments can be used:

```javascript
const APPLICATION_FRAGMENT = gql`
  fragment ApplicationFields on Application {
    _id
    teamName
    ref
    contactEmail
    contactPhone
    country
    createdAt
    updatedAt
  }
`;

const GET_APPLICATION = gql`
  query GetApplication($accessToken: TokenInput!, $applicationId: ID!) {
    getApplication(accessToken: $accessToken, applicationId: $applicationId) {
      ...ApplicationFields
      studentRecords {
        firstName
        lastName
        email
      }
      projectRecords {
        title
        category
      }
    }
  }
  ${APPLICATION_FRAGMENT}
`;
```

## Best Practices

1. **Use Fragments**: For reusable query parts
2. **Handle Loading States**: Show loading indicators
3. **Handle Errors**: Provide clear error messages
4. **Optimize Queries**: Request only needed fields
5. **Use Cache**: Leverage Apollo's caching system
6. **Implement Optimistic UI**: For better user experience
7. **Secure Mutations**: Validate input on both client and server

