---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Updating the cache after a Mutation
details:
  section: 5
  lesson: 31
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



### Update Cache After Mutations 

In the previous example we saw how we can take individual resolvers and ensure that they data they fetch will never get cached. Although that approach works and may in fact be the right approach in some use cases, more often then not we'll need a more versatile caching strategy. 

> In our case, our data and functionality implemented thus far allow for us to reduce the request calls because we're actually already sending back the data we need. 
>
> We have two queries that are almost identical currently, and can easily be made to reflect the data necessary
>
> ![image-20201017163150376](https://tva1.sinaimg.cn/large/007S8ZIlly1gjszr73yk9j30qo0awq56.jpg)
>
> - only the description field is not included with the `CreateJob` mutation, we can simply update that and add the description field so that the response from creating a new Job, gives us the exact response required by `loadJob()`



Once we've made sure we're querying the exact fields we need, we can update our mutation to allow apollo-client to have access to the response, using the provided `update` function. 

```js
// requests.js

update: (cache, mutationResult) => {
  console.log("createJob -> cache, mutationResult", {
// @param: cache - store / props provided by apollo -- simple object to use as storage
    cache,
// @param: result - the response or "result" received from our mutation
    result,
  });
```

if we take a look at the console after adding a new post we'll have a better idea of what each of these arguments is:

> **InMemoryCache**
>
> is a reference to our apollo cache 
>
> ![image-20201017172256997](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt18dtq3oj30vu072gop.jpg)



> **result**
>
> literally the result from our mutation
>
> ![image-20201017172343927](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt196ep9hj30r00673zs.jpg)



Now that we know that our job is included with the result we can just destructure what we need from the mutation result inline:

```js
update: (cache, { data: { job } }) => { /* handle mutation results here */ },
```



Now in order to pass the right information to our cache, we'll need to extract the `JobQuery` from the `loadJobs()` function, and us that in our `createJob()` result to set our context with:

```js
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


export async function loadJob(id) {
  // 🚧 refactor extract query to be shared with createJob()

  // const query = gql`
  //   query JobQuery($id: ID!) {
  //     job(id: $id) {
  //       id
  //       title
  //       company {
  //         id
  //         name
  //       }
  //       description
  //     }
  //   }
  // `;

  const {
    data: { job },
    // now uses extracted `jobQuery`
  } = await client.query({ query: jobQuery, variables: { id } });

  return job;
}


export async function createJob(input) {
  const mutation = gql`
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
  if (!input) throw new Error("please provide a valid id");
  const {
    data: { job },
  } = await client.mutate({
    mutation,
    variables: { input },
    //📔  update function gets called after a mutation has executed
    update: (cache, { data }) => {
          cache.writeQuery({
            // use extracted query from `loadJobs()` to store job to cache on update
            query: jobQuery, 
            // requires 1 arg, an id, that it uses to find a job
            variables: { id: data.job.id }, 
            // the data value we get from the mutation result passed into this function as `data`
            data, 
          });
        },
  });
```

With this in place we are no writing the returned job to our cache by requerying for the newly created job by leveraging the same query we use for the `loadJob()`  function. In the end the result is that instead of passing in the default behavoir we're giving apollo the exact data we'd like to make sure it adds to the cache for us. So instead of avoiding cache, we're now declaratively choosing what data needs to be cached and when -- then we inform apollo of that new data with the update function 

![checkrequests](https://tva1.sinaimg.cn/large/007S8ZIlly1gjt3dp9g86g30l60q2u0x.gif)

We can verify this change in the browser like before, we expect that once we create a new job, our application should get the data for the new item stored into its cache directly via our update function. So although the jobboard will in fact still query for new data when we go to it, our new item will appear in the list and when clicked it will not make the server make a new request, instead the request is being served from the cache input that we provided in our mutation update function. 