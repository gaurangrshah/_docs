---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Apollo Client Setup
details:
  section: 5
  lesson: 27
---



- [ ] install [`apollo-boost`](https://www.apollographql.com/docs/react/v2.4/advanced/boost-migration/) on the client-side of our application

  ```shell
  npm install apollo-boost graphql
  ```





# Apollo Client Setup

[Introduction to Apollo Boost](https://www.digitalocean.com/community/tutorials/graphql-apollo-boost-introduction)

Currently we've written our own request function and made our requests using the fetch api, although this can get cumbersome and there are better ways to handle this functionality as well as getting a lot more in the process using `apollo-boost` to process our requests. If we take a look at the apollo-boost package and it's dependencies we'll see that all it really is - is a bundle of Apollo packages that we'll need in our project:

```json
➜ npm view apollo-boost dependencies
 
{
  'apollo-cache': '^1.3.5',
  'apollo-cache-inmemory': '^1.6.6',
  'apollo-client': '^2.6.10',
  'apollo-link': '^1.0.6',
  'apollo-link-error': '^1.0.3',
  'apollo-link-http': '^1.3.1',
  'graphql-tag': '^2.4.2',
  'ts-invariant': '^0.4.0',
  tslib: '^1.10.0'
}
```

> All of these libraries are required to use apollo-client and so Apollo-boost is just an easy way to get all of them in one shot. 



Our first step will be to configure our client:

```js
// reqeusts.js

import { ApolloClient, HttpLink, InMemoryCache } from "apollo-boost";


// create a new instace of apollo-client
const client = new ApolloClient({
  // client config options
  link: new HttpLink({ uri: devEndpointURL }), // sets the endpoint
  cache: new InMemoryCache(), // using InMemoryCache implementation
});

```

> [`HttpLink`](https://www.apollographql.com/docs/react/networking/advanced-http-networking/#the-httplink-object)
>
> Apollo Client uses `HttpLink` to send GraphQL operations to a server over HTTP. The link supports both POST and GET requests, and it can modify HTTP options on a per-query basis. This comes in handy when implementing authentication, persisted queries, dynamic URIs, and other granular updates.
>
> [`InMemoryCache`](https://www.apollographql.com/docs/react/api/cache/InMemoryCache/#gatsby-focus-wrapper)
>
> Run GraphQL queries directly against the cache.
>
> For usage instructions, refer to [Interacting with cached data: `readQuery`](https://www.apollographql.com/docs/react/caching/cache-interaction/#readquery).



Now we can make use of this new functionality in our requests by replacing our own graphqlRequest implementation:

```js
// requests.js

import gql from 'graphql-tag' // query helper


export async function loadJobs() {
  // wrap the query in the `gql tag function` which parses it as a query
  // this is the format expected by apollo-client, and must be named "query"
  const query = gql`
    query JobsQuery {
      jobs {
        id
        title
        company {
          id
          name
        }
      }
    }
  `;

  // 🚧 refactor : replace with ApolloClient request:
  // const data = await graphqlRequest(jobsQuery);
  // use apollo-client to process our request query
  const {
    data: { jobs },
  } = await client.query({ query }); // ❗️ the "query" name is required to be query!!!
  // destructure jobs from the destructured data property
  // there are several arguments available here,
  // including errors and data, we only need data
  
  console.log("hi im a request from apollo", { jobs });
  
  return jobs;
}
```

> - `gql` A JavaScript template literal tag that parses GraphQL query strings into the standard GraphQL AST.
> - `apollo-client` uses gql tags as its format of processing queries



With this in place we should be able to see our job list now being process by apollo-client.

![image-20201017031731999](https://tva1.sinaimg.cn/large/007S8ZIlly1gjscsqfdaqj30tv0uzdk5.jpg)



Now let's take a look at an example that takes in some variables:

```js
// requests.js

export async function loadJob(id) {
  // ensure we use the gql tag and that the query variable name is query
  const query = gql`
    query JobQuery($id: ID!) {
      job(id: $id) {
        id
        title
        company {
          id
          name
        }
        description
      }
    }
  `;
  // 🚧 refactor with apollo-client
  // const { job } = await graphqlRequest(jobQuery, { id });
  const {
    data: { job },
    // note: we can pass variables in as an object with the variabels property
  } = await client.query({ query, variables: { id } });
  return job;
}
```

> in this example we were able to refactor our own graphqlReqest function with apollo's client and incorporate our variables required to display each individual job.



In this way we can refactor each of our request functions:

```js
// requests.js

export async function loadCompany(id) {
  // const companyQuery = `query CompanyQuery($id: ID!){
  const query = gql`
    query CompanyQuery($id: ID!) {
      company(id: $id) {
        id
        name
        description
        jobs {
          id
          title
          description
        }
      }
    }
  `;

  // const { company } = await graphqlRequest(companyQuery, { id });
  const {
    data: { company },
  } = await client.query({ query, variables: { id } });
  return company;
}
```



And finally we can refactor our `createJob()` mutation:

```js
// resolvers.js

export async function createJob(input) {
  // rename mutation to match the mutation property expected by apollo-client
  const mutation = gql`
    mutation CreateJob($input: CreateJobInput) {
      job: createJob(input: $input) {
        id
        title
        company {
          id
          name
        }
      }
    }
  `;
  if (!input) throw new Error("please provide a valid id");
  // 🚧 refactor with apollo-client
  // const { job } = await graphqlRequest(CreateJobMutation, { input });
  const {
    data: { job },
  } = await client.mutate({ mutation, variables: { input } });
  // using the mutate function from apollo-client to process our mutations.
  return job;
}
```

> - [`mutate()`](https://www.apollographql.com/docs/react/api/core/ApolloClient/#ApolloClient.mutate)
>
> This resolves a single mutation according to the options specified and returns a[Promise](https://www.apollographql.com/docs/react/api/core/ApolloClient/#Promise) which is either resolved with the resulting data or rejected with an error.





There's still one more step, which is to provide our authorization headers for our `createJob()` mutation. Initially our original implementation set the headers dynamically if the user was logged in, we'll need to do the same for our Apollo-client requests. Because of this our front-end requests will currently come back as `Unauthorized`:

![image-20201017033442249](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsdallcz8j30rw08sq4c.jpg)





In order to achieve similar logic we can use another helper from our apollo-client `ApolloLink`:

```js
// requests.js

const devEndpointURL = "http://localhost:9000/graphql";

const authLink = new ApolloLink((operation, forward) => {
  if (isLoggedIn()) {
    // set context with headers if user is loggedIn
    operation.setContext({
      // operation has a context from which we can set our headers
      headers: { authorization: "Bearer " + getAccessToken() },
    });
  }
  return forward(operation);
});

// create a new instace of apollo-client
const client = new ApolloClient({
  // client config options

  link: ApolloLink.from(
    // allows setting of multiple apolloLink configurations as an array or ApolloLinks
    [
      authLink, // include authLink before HttpLink to ensure we're able to set headers first
      new HttpLink({ uri: devEndpointURL }),
      // sets the endpoint and uses authLink headers for request if defined
    ]
  ),
  cache: new InMemoryCache(), // using InMemoryCache implementation
});
```

> - [`ApolloLink`](https://www.apollographql.com/docs/link/composition/#gatsby-focus-wrapper)
>
> Apollo Link ships with two ways to compose links. The first is a method called `from`which is both exported, and is on the `ApolloLink` interface. `from` takes an array of links and combines them all into a single link. 
>
> Here ApolloLink gives us two arguments:
>
> - [`operation`] - specifies the operation or action to take
> - [`forward`] -  function use to move to the next operation (when many exist)
>
> We use `ApolloLink` as `authLink` to create and set the loggedIn user request headers only if a user is authenticated, if so we inser the headers into our `HttpLink` using `ApolloLink.from()`
>
> This handles the request dyanmically passing in our headers via the operation context if our user is loggedIn and authenticated.

