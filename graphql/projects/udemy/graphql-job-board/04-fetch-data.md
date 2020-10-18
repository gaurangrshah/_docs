---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Fetching Jobs in the Client
details:
  section: 2
  lesson: 15
---



# Fetching Jobs in the Client

Now with our main dataTypes and their relationship setup, we're ready to query for our data from our client-side react application.

![image-20201016003950917](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr2mctupnj30xj0aidg9.jpg)

```shell
cd client && npm run start
```

> **NOTE:** we already have several users with credientails in our mock database, that we ca use in order to login to the clientside application. 





```js
const devEndpointURL = "http://localhost:9000/graphql";

export async function loadJobs() {
  console.log("running load jobs");
  const response = await fetch(devEndpointURL, {
    method: "POST",
    headers: { "content-type": "application/json" },
    // our query is inserted into the request body:
    body: JSON.stringify({
      query: `{
				jobs {
					id
					title
					company {
						id
						name
					}
				}
			}`,
    }),
  });
  const responseBody = await response.json();
  console.log({ responseBody });
  // graphql response has a data property which we can append our query results onto
  return responseBody.data.jobs;
}

```

> **NOTE:** that we've only queried for the data our application needs, our front end is not making use of the description field on either the company or the job data, so we've simply omitted that from our request. 



Now we can import his function into our <JobBoard/> component, and replace the dummy data with the graphql data:

```js
import React, { Component } from "react";
import { JobList } from "./JobList";
// const { jobs } = require('./fake-data');
import { loadJobs } from "./requests";

export class JobBoard extends Component {
  constructor(props) {
    super(props);
    // instantiate state to hold jobs
    this.state = { jobs: [] };
    // due to the asynchonous nature of requests, we want to ensure
    // we're setting an initial state, because our component will not
    // intially load with any data, instead will get the data fetched from our loadJobs() request
  }

  async componentDidMount() {
    // asynchronously load jobs and setState
    const jobs = await loadJobs();
    console.log(jobs);
    this.setState({ jobs });
  }

  render() {
    // destructure jobs from state
    const { jobs } = this.state;
    return (
      <div>
        <h1 className='title'>Job Board</h1>
        {/* pass jobs into <JobList />  where they are rendered */}
        <JobList jobs={jobs} />
      </div>
    );
  }
}

```

![image-20201016011908463](/Users/bunty/Library/Application Support/typora-user-images/image-20201016011908463.png)