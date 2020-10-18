---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Graphql Fragments
details:
  section: 5
  lesson: 32
---



## Graphql Fragments



As we saw in the last lesson it can be beneficial to make our graphql queries re-usable. Up till now we've defined them as local variables in each of our functions, instead we'll want to extract our queries to the top level of our file so that we can make re-use them if needed. 



```js
// requests.js

const createJobMutation = gql`
  mutation CreateJob($input: CreateJobInput) {
    job: createJob(input: $input) {
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

const jobQuery = gql`
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

const jobsQuery = gql`
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

const companyQuery = gql`
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
```

> - here are all of our queries extracted out of their local functions
>
> - and the functions using these queries now look like:
>
> ```js
> export async function createJob(input) {
> 
>   if (!input) throw new Error("please provide a valid id");
>   const {
>     data: { job },
>   } = await client.mutate({
>     mutation: createJobMutation,
>     variables: { input },
>     update: (cache, { data }) => {
>       cache.writeQuery({
>         query: jobQuery,
>         variables: { id: data.job.id },
>         data,
>       });
>     },
>   });
> 
>   return job;
> }
> 
> export async function loadCompany(id) {
> 
>   const {
>     data: { company },
>   } = await client.query({ query: companyQuery, variables: { id } });
> 
>   return company;
> }
> 
> export async function loadJob(id) {
> 
>   const {
>     data: { job },
>   } = await client.query({ query: jobQuery, variables: { id } });
> 
>   return job;
> }
> 
> export async function loadJobs() {
>   const {
>     data: { jobs },
>     // "no-cache" will not cache this request EVER!
>   } = await client.query({ query: jobsQuery, fetchPolicy: "no-cache" });
> 
>   return jobs;
> }
> 
> ```





Now with this in place the functionality of our current code hasn't changed at all, but our code is all the more re-usable now. 



Graphql provides a similar type of feature called `fragments` the act similar to how inputs work with mutations. Fragments allow us to define the data we expect back from a query and then re-use that frament as a shorthand to represent that default group of fields each time. We can re-use our fragments with any applicable query. 

Let's take a look at how we can work with fragments in our playground

![image-20201017191038079](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt4cfj2ptj30v20g6tbc.jpg)

> A frament can be through of as a `query fragement` its a grouping of the exact fields we need for a given query the fragement is then used in place of the fields in our queries with a similar syntax to the javascript spread syntax.



Now let's take a look at how we can use fragments for our frontend requests:

```js
// requests.js

const jobDetailFragment = gql`
  fragment JobDetail on Job {
    id
    title
    company {
      id
      name
    }
    description
  }
`;
```



Now we can re-use this fragment anywhere we need to query for job details:

```js
const createJobMutation = gql`
  mutation CreateJob($input: CreateJobInput) {
    job: createJob(input: $input) {
      ...JobDetail # fragment gets used here
    }
  }
  # fragment definition gets inserted here
  ${jobDetailFragment}
`;
```

> **NOTE: ** although this looks like we're passing in a string interpolation for `jobDetailFragment` this is not actually the case we're actually passing in an entire tag function as our fragment is wrapped in a `gql` tag. It is resolved inline as an expression in our template literal, and injects our fields into the query through the speaded `JobDetail` fragement reference.

We can also use the same fragment in our `jobQuery`

```js
const jobQuery = gql`
  query JobQuery($id: ID!) {
    job(id: $id) {
      ...JobDetail
    }
  }
  # fragment definition gets inserted here
  ${jobDetailFragment}
`;
```



Now if we test our frontend, we should see our standard request queries replaced with our fragments:

![image-20201017192209579](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt4oeomf6j30r9079tb1.jpg)

