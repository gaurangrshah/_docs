---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Mutation Best Practices
details:
  section: 3
  lesson: 22
---



## Mutation Best Practices

We'll learn how to make our mutations more viable.



To start we'll update our schema, mutation and resolver to expose the entire job object to our createJob mutation so instead of just getting the jobId of the newly created job we will have access to the entire new Job object

```
# schema.graphql
# createJob(companyId: ID, title: String, description: String): ID
createJob(companyId: ID, title: String, description: String): Job
```

> return the entire job as a response to our `createJob` request

```js
// resolvers.js

const Mutation = {
  // 2nd arg destructures the values from "args"
  createJob: (root, { companyId, title, description }) => {
    // creating a job will return our new jobId
    const id = db.jobs.create({ companyId, title, description });
    // use jobId to query for the job data
    return db.jobs.get(id);
  },
};
```

> here instead of immediately returning just the id, we use the id to query for the newly created job object and return the entire job object

![image-20201016183639598](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrxqrsotvj316w0b9wg4.jpg)

> **❗️❗️❗️❗️❗️ALIASING REQUEST PROPERTIES**
>
> **NOTE**: u can rename (aka alias) our mutation name (and query names) in our graphql request, which will then be applied to the object in the response. Therefore here we've renamed `createJob()` to `job` and so if you look at the response, you'll notice that its not `createJob()` that gets appended to `data`, but rather our `alias` for `createJob`





### Passing Mutation arguments via variables:

There are a few ways to achieve this, one of the more cumbersome is to define all the values that we expect for our mutation. 

> **❗️ALSO** rememeber to name any queries or muations that take arguments, it is not optional. 
>
> ![image-20201016184533957](https://tva1.sinaimg.cn/large/007S8ZIlly1gjry014x4fj30y50cj40y.jpg)



The above method works, but is very cumbersome on the client side, so we have another way to simplify it by using a graphql `input` type, <u>arguments can only be defined by **input types**.</u>

```
# schema.graphql

input CreateJobInput {
	companyId: ID
	title: String
	description: String
}
```

> - it is a convention to name the input types to correspond with where they are expected to be used 
> - in our case the `createJobInput` is used for the `createJob` mutation



Now we can use the newly created `createJobInput` type to replace our 3 expected arguments on our mutation:

```
#schema.graphql

type Mutation {
	## the mutation now uses the params defined in our createJobInput
  createJob(input: CreateJobInput): Job
}
```

> **Input and output types**
>
> [source](https://atheros.ai/blog/input-object-type-as-an-argument-for-graphql-mutations-and-queries)
>
> According to [GraphQL specification](https://facebook.github.io/graphql/), when we deal with its type system, we have to discuss two different categories of types:
>
> - **output type** can be used for definition of data, which is obtained after query execution;
> - **input types** are used as a query parameters, e.g., payload for creating a user. In graphql-js library we basically have two different types, which can be used as objects. **GraphQLObjectType** (an output type), and **GraphQLInputObjectType** (an input type).



Now we must also update our resolver:

```js
// resolvers.js

const Mutation = {
  // createJob: (root, { companyId, title, description }) => {
  createJob: (root, { input }) => {
      // receives a single input argument from our mutation as args

		const id = db.jobs.create(input); // createJobInput

    // use jobId to query for the job data
    const job = db.jobs.get(id);
    console.log("🎯", { job });
    return job;
  },
};
```



Now we can test our input in the playground:

![image-20201016192439450](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrz4pfltbj30uq0grjtn.jpg)





### Handling mutations on the client side

```js
// requests.js

export async function createJob(input) {
  const CreateJobMutation = `mutation CreateJob($input: CreateJobInput) {
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

  const { job } = await graphqlRequest(CreateJobMutation, input);
  return job;
}
```

```js
// JobForm.js

import { createJob } from "./requests";

handleClick(event) {
  event.preventDefault();
  const companyId = "HJRa-DOuG"; //FIXME: temporarily hard-coding the companyId:
  createJob({ companyId, ...this.state }).then((job) => {
    // use react router to push a new path into the browser:
    this.props.history.push(`/jobs/${job.id}`);
  });
  console.log("should post a new job:", this.state);
}
```

