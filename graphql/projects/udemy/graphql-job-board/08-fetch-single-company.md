---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Fetching a Company by ID
details:
  section: 2
  lesson: 19
---



# Fetching a company by id



Currently our application has no way of directly querying for a company. Company details are only accessible through their relationship to a Job. So in order to expose an endpoint for our companies, we'll need to update our schema and our resolvers:

```
# schema.graphql

type Query {

# company(id) => Company(query with id returns matching company)
	company(id: ID!): Company 
	
  job(id: ID!): Job
  jobs: [Job]
}
```

> **NOTE:** the company query and the job query are identical in their logic, therefore their resolvers will also resemble eachother:

```js
// resolvers.js

const Query = {
  
  company: (root, args) => db.companies.get(args.id), // uses id from args to find matching company
  
  job: (root, args) => db.jobs.get(args.id), 
  jobs: () => db.jobs.list(),
};
```

Now we can test our query using one of our companyIds:

![image-20201016123723272](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrncxzci3j30te0blwfr.jpg)



Now we can update the request logic client-side application with our new functionality:

```js
// requests.js

export async function loadCompany(id) {
  const companyQuery = `query CompanyQuery($id: ID!){
    company(id: $id) {
      id
      name
      description
    }
  }`;

  const { company } = await graphqlRequest(companyQuery, { id });
  return company;
}
```

Finally we can replace the dummy data in the front-end with our new api request data:

```js
// CompanyDetail.js

import React, { Component } from "react";
// import { companies } from './fake-data';
import { loadCompany } from "./requests";

export class CompanyDetail extends Component {
  constructor(props) {
    super(props);

    this.state = { company: null };
  }

  async componentDidMount() {
    // fetch company with companyId
    const { companyId } = this.props.match.params;
    const company = await loadCompany(companyId);
    this.setState({ company }); // set state with reponse
  }

  render() {
    const { company } = this.state;
    if (!company) {
      // show nothing if no company
      return null;
    }
    return (
      <div>
        <h1 className='title'>{company.name}</h1>
        <div className='box'>{company.description}</div>
      </div>
    );
  }
}
```

