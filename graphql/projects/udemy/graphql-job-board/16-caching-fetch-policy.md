---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Caching and Fetch Policy
details:
  section: 5
  lesson: 30
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---





# Caching And Fetch Policy

Now that we've refactored our application to use Apollo client and have successfully added authentication to our application, by setting the user details to the apollo context, thus giving access to user info from any resolver. 

That's all well and good, but what difference did that make for our application aside from authorization context? Everything pretty much works the same as it did before with our fetch api implementation. 



**Caching**

One of the core features of apollo-client is it's caching, and the best way to observe how this works is by doing some good old fashioned network debugging. So below we've set a filter to only show our graphql requests, we're also logging to the console every time one of our resolvers gets called:

> ![cache-example](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsz6452yug30m90rdkjo.gif)
>
> **NOTE:** our api is very small, and so there's not much data to pull,  but the who example shows that our resolvers do not have to keep making requests once apollo has the data, if apollo already has the data we need in it's cache it will serve us the data without having to go back to the server. 
>
> Anytime a new call is made it will make the trip like when we first open up each individual job page. But if we go back to that same job, there will be no server call again. 
>
> This is the power of apollo's caching. 



**Downside**

Caching helps us reduce server calls, but sometimes can lead to unexpected behavior. In the example above we make a post request to create a new Job, when we do the application actually makes two separate calls to our server, one to post the request and another to fetch the the new job details. 

This second call is also unnecessary, because we actually send the job details back as a response from our `createJob()` mutation, so the client shouldn't have to make that second call. Apollo-client doesn't cache this response data by default, because it's the result of a mutation -- this is it's default behavior.

> ⬇️ we'll be taking care of this un-needed behavoir in the next lesson. 

Another side-effect is that when we get taken back to the <JobBoard /> our newly created <Job/> doesn't appear, this is because apollo is loading the data it already has in it's cache, and did not make a new request to get the new data to update our application. Refreshing the page will trigger a new request and that will pull in the data. This is also the default behavior, but it is also not what we want. 



To fix this, we can actually bypass the cache completely when executing the `loadJobs()` function

```js
// requests.js

export async function loadJobs() {
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

  const {
    data: { jobs },
    
    // 🔵 add a cache fetch policy default is "cache-first"
    // "no-cache" will not cache this request EVER!
    
  } = await client.query({ query, fetchPolicy: "no-cache" });
  
  return jobs;
```

Now anytime we go trigger the `loadJobs()` function, our application will make a new request. Essentially this particular request is opting out of the caching strategy that apollo-client provides. This ensures that the data tha appears on this page will always be in sync with the server. 

When used sparigly this can help increase user satisfaction, but keep in mind that in order to keep your application agile, you should minimize as many requess as possible. 



While this approach works, and can be needed in some cases, we can look at a more flexible way of achieving this in the next lesson...

