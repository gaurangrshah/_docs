---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Query Variables: Fetch a Job
details:
  section: 2
  lesson: 17
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



## Fetch a Single Job By Id



currently our <JobDetail/> is expecting its data from our fake-data api, we'll need query for the data it needs using our newly created resolver. 

```js
// JobDetail.js

import React, { Component } from 'react';
import { Link } from 'react-router-dom';
import { jobs } from './fake-data';

export class JobDetail extends Component {
  constructor(props) {
    super(props);
    const {jobId} = this.props.match.params;
    this.state = {job: jobs.find((job) => job.id === jobId)};
  }

  render() {
    const {job} = this.state;
    return (
      <div>
        <h1 className="title">{job.title}</h1>
        <h2 className="subtitle">
          <Link to={`/companies/${job.company.id}`}>{job.company.name}</Link>
        </h2>
        <div className="box">{job.description}</div>
      </div>
    );
  }
}

```



As we can see in the above component we'll need the following data from our query:

![image-20201016015415242](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr4rrsgxjj30n50a4my8.jpg)



Now let's use this query to create a `loadJob()` that will query for our job by it's id in the url params



> **NOTE:** We need to pass an id as a variable into our query, and instead of using a template literal when we already have utilized for our query we can take advantage of graphql's solution for passing along query variables.



> 💡 it's typically a great idea to name your queries - this allows the console to provide better debug messages, and helps to find errors faster. 
>
> ![image-20201016015918460](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr4x11y1aj30sz0b2q45.jpg)
>
> ❗️Also note when passing variables, the `query` "operator" is required whether or not you name the query. 





**Passing Variables into a Query**

It's important to note that query variables must be typed in place:

![image-20201016020725672](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr55hcsu9j30su0a4jsg.jpg)

```
query JobQuery($id: ID!){ # variable definition and type
  job(id: $id) { # variable being passed in as argument to query
    id
    title
    company {
      id
      name
    }
    description
  }
}
```

> the variable `$id` is of the type `ID!` and the (`!`) means its non-nullable and therefore: requried. Now we have a parameter `$id` that we can can pass our value into with our query.



We can test query variables in the playground, in the bottom panel. These values as sent as a standard JSON object. Here's what our query variable object would look like: 

```json
{
  "id": "rJKAbDd_z"
}
```

And if we execute the query we should get the same results as our original hard-coded example:

![image-20201016020939161](/Users/bunty/Library/Application Support/typora-user-images/image-20201016020939161.png)

And now if we update the variable with a new id, we should get another result:

![image-20201016021038724](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr58sx8htj30rk0apjsk.jpg)





```js
// requests.js

export async function loadJob(id) {
  // id is passed into loadJob
  console.log("running load jobs");
  const response = await fetch(devEndpointURL, {
    method: "POST",
    headers: { "content-type": "application/json" },
    // our query is inserted into the request body:
    body: JSON.stringify({
      query: `query JobQuery($id: ID!){
        job(id: $id) {
          id
          title
          company {
            id
            name
          }
          description
        }
      }`,
      variables: { id }, // 2nd prop on Json request gets the variable
    }),
  });
  const responseBody = await response.json();
  console.log({ responseBody });
  // graphql response has a data property which we can append our query results onto
  return responseBody.data.job;
}
```

> **NOTE:** the query gets passed in as is, the variables property is available as a 2nd argument, we place our id value onto it, and it treats it exactly as our mock query in the playground client.  

![query-variable.gif.sb-50704d86-qGyLjZ](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr5qz4iiog30to0r8kjz.gif)





### Making Queries Re-usable

As long as we're querying the same api our endpoint does not change, most of the usage of our query will actually get repeated. In order to avoid this we'll make our query more re-usable:

```js
// requests.js

export async function graphqlRequest(query, variables = {}) {
  // @param: query = graphql query * required
  // @param: variables = graphql varaibale arguments (optional)
  const response = await fetch(devEndpointURL, {
    method: "POST",
    headers: { "content-type": "application/json" },
    // query and variables passed into request body as JSON
    body: JSON.stringify({ query, variables }),
  });
  const responseBody = await response.json();
  // return the generic root data object directly
  return responseBody.data 
  // return responseBody.data.job;
}
```

> Now both the query and optional variables can be passed into our `graphqlRequest()` as arguments



Then we can use this `graphqlRequest()` to now replace the logic in our current `loadJob()` and `loadJobs()` functions:

```js
// requests.js

export async function loadJob(id) {
  const jobQuery = `query JobQuery($id: ID!){
        job(id: $id) {
          id
          title
          company {
            id
            name
          }
          description
        }
      }`;
  const { job } = await graphqlRequest(jobQuery, { id });
  return job;
}


export async function loadJobs() {
  const jobsQuery = `query JobsQuery{
				jobs {
					id
					title
					company {
						id
						name
					}
				}
			}`;
  const { jobs } = await graphqlRequest(jobsQuery);
  return jobs;
}
```

> We've pretty much extracted all of the logic into our reusable `graphqlRequest()`, our `loadJob()` and `loadJobs()` functions are only responsible for providing the request with the right query and arguments to get back the correct data. 

