---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Returning an Array of Jobs
details:
  section: 2
  lesson: 12
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



### Setting up typeDefs for a Job 

```
# schema.graphql

type Query {
  jobs: [Job]
}
```

> Here we've updated our query to state that we expect the jobs query to return an array of items of the type `Job`  --- so essentially an array of jobs.



```
# schema.graphql

type Job {
  id: ID! 
  title: String
  description: String
}
```

> **❗️NOTE**  (exclamation operator) "!" == not nullable/required



```js
//resolvers.js

const Query = {
  jobs: () => [], // returning an empty array for testing purposes
};

module.exports = { Query };
```

![image-20201015220318566](https://tva1.sinaimg.cn/large/007S8ZIlly1gjqy3h1vdcj30r607574o.jpg)

> As expected we get back our empty array from our resolver. 





```js
// resolvers.js

const db = require("./db"); // using "notarealdb" to mock a db

const Query = {
  jobs: () => db.jobs.list(),   // list() is from notarealdb
};

module.exports = { Query };
```

> This time we get back our mock data:
>
> ![image-20201015220639416](https://tva1.sinaimg.cn/large/007S8ZIlly1gjqy6yz0w8j30rm0aoaba.jpg)

