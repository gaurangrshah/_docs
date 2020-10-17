---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Create a Job
details:
  section: 3
  lesson: 21
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



## Create a Job Mutation

> ❗️**NOTE:** you'll need to login with the provided credentials from the users data

![image-20201016154406348](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrsr84ltej30tm0ee0t3.jpg)

Currently our front-end form does not actually talk to our server, we'll need to change that, but before we can we'll need to update our schema but this type we'll need to utilize a new type called `mutations`.

> `mutations` mutate or change our data. while `resolvers` are used to simply resolve our data, mutations will more-than-likely need to take arguments (the data we need to update). In order to create a new job we'll need to create a mutation to handle that task for us. We'll stil need a resolver to help resolve the data that gets returned by our mutation - so they work together in that way. 



```
# schema.graphql

type Mutation {
  createJob(companyId: ID, title: String, description: String)
}
```

> `mutations` are a root type just like `Query`
>
> ❗️**NOTE:** we did not require an id param in our mutation, this is because we want the server to handle that for us -- we do not want to hardcode it or overwrite the server's id generating functionality.

```js
// resolvers.js

const Mutation = {
  // 2nd arg destructures the values from "args"
  createJob: (root, { companyId, title, description }) => {
    // creating a job will return our new jobId
    return db.jobs.create({ companyId, title, description });
  },
};


module.exports = { Query, Mutation, Job, Company }; // export new mutation
```



Now we can test our Mutation:

![image-20201016155713416](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrt4vi11kj30yg07mwf6.jpg)

We get back the expected newy created `jobId`, we can also verify that a new job was added to our front end as well:

![image-20201016155829919](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrt66fvwhj30q4093dg8.jpg)



