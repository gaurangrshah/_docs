---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Arguments: Returning a Job By Id
details:
  section: 2
  lesson: 16
---



## Fetching Job Details

In order to fetch job details we'll need to return our corresponding job by the `id` property:

- We currently have data in our <JobList/>, but our <JobDetail/> is still looking for it's data through our dummy data, we'll need to fetch our job details and replace the dummy data



> **NOTE:** we have an route already defined that uses our <JobDetail/>

```js
// app.js

<Route path="/jobs/:jobId" component={JobDetail} />
```



**Update Resolvers to handle Job By Id**

currently we don't have any resolvers that can handle this endpoint, so we'll need to create one, that will be able to take a jobId as an argument from our url parameters.

```js
// schema.graphql

type Query {
  // requires the id, of type ID, which is (!) non-nullable and required
  job(id: ID!) : Job # job(id) => Job (query with id, returns a matching Job) 
  jobs: [Job]
}
```

```js
// resolvers.js

const Query = {
  // uses id from args to find matching job
  job: (root, args) => db.jobs.get(args.id), 
};
```

> **NOTE:** the first argument, `parent` is now the `root` object, which is our `Query` type itself. Just `Job` type is the parent of the `Company`, here the `root`  is the `parent` of `Job`

![image-20201016013332296](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr4684hwmj30nn096q3p.jpg)



**GraphQL Arguments: 'args'**

`args` => is the 2nd argument in a graphql resolver. 

> This gives us access to the arguments passed into our queries. In the above example we are able to access the job by th id passed in via `args`.





> **NOTE:** we can also destructure the items we need from args in place as well:
>
> ```js
> // resolvers.js
> 
>   job: (root, { id }) => db.jobs.get(id), 
> ```

