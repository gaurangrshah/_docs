---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Returning Jobs for a Company
details:
  section: 2
  lesson: 20
---



# Returning Jobs for a Company

We'll want to implement a new feature where we can show a list of currently available job positions when someone navigates to our company details page. Currently none of our queries allows for this behavior so we'll start by updating our schema and resolvers:

```
# schema.graphql

type Company {
  id: ID!
  name: String
  description: String
  jobs: [Job] # jobs represents an array of Job items
}
```

> ❗️ **REMEMBER** it is essential to retain an close 1:1 mapping between our typeDefs and our resolvers. So it's important to note that our `jobs` field is current defined in our `Company` type as a field. Our resolver will need a Company object to mimic this `typeDef`

```js
// resolvers.js

const Company = {
  // access the parent company object via args
  jobs: (company) =>
    db.jobs.list().filter((job) => job.companyId === company.id),
  // use the parent company to find jobs by companyId from the jobs list
};

module.exports = { Query, Job, Company }; // remember to export any resolver objects
```



> 🤔 **NOTE: ** the difference between our jobs and company type resolvers, this is because Job has a one-to-one relationship with company while, company has a one-to-many relationship to jobs. 
>
> i.e., A company can have many jobs, but a job can only belong to one company. 



Now lets test our query in the playground:

![image-20201016151234307](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrruf0ccij30st0e8q4w.jpg)





then we can move onto the client side and ensure update our query and handle the new data:

```js
// requests.js


export async function loadCompany(id) {
  const companyQuery = `query CompanyQuery($id: ID!){
    company(id: $id) {
      id
      name
      description
      jobs { ## update company query to grab related jobs
        id
        title
        description
      }
    }
  }`;

  const { company } = await graphqlRequest(companyQuery, { id });
  console.log(company.jobs); // confirm we're getting all related jobs
  return company;
}
```

```js
// CompanyDetail.js

return (
      <div>
        <h1 className='title'>{company.name}</h1>
        <div className='box'>{company.description}</div>
        {/* print jobs if available */}
        {company?.jobs && <pre>{JSON.stringify(company.jobs)}</pre>}
      </div>
    );
  }
```

![image-20201016151857803](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrs121wwlj30pz06zaaj.jpg)

```js
// CompanyDetail.js

return (
      <div>
        <h1 className='title'>{company.name}</h1>
        <div className='box'>{company.description}</div>

        <h5 className='title is-5'>Jobs At: {company.name}</h5>
        <JobList jobs={company.jobs} />
      </div>
    );
```

![image-20201016152146277](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrs3yt683j30tn0c8mxs.jpg)



### GraphQL Nesting 

Once we have certain relationships established in our type association and we've setup the resolvers accordingly. Then graphql makes querying for related data a breeze. Below is a contrived example to show each part of our company and job relationship is able to reference itself and its parent over and over again. Again this is a contrived example, but it shows how you can ask for any data you want from any related types:

![image-20201016153001707](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrsckxpkyj30yg0u0n1z.jpg)