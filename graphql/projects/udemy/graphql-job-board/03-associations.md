---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Object Asscoiations 
details:
  section: 2
  lesson: 14
---



## Object Associations

Object associations basically allow us to query for related data. By associating our objects we're achieving a relational mapping.

Currently we have a single type of `Job`, we're going to now add another type that we'd like associated with our `Job`, the `Company` type.

```
# schema.graphql

type Company {
  id: ID!
  name: String
  description: String
}
```



Now all we have to do to create a relation is to give our `Job` type a relation to our `Company` type:

```
#schema.graphql

type Job {
  id: ID!
  title: String
  company: Company # creates an association, to the company type
  description: String
}
```



> **NOTE:** Remember that we always must reflect the changes in our typeDefs with resolvers to handle those queries. If we fail to make the update on both ends, we will simply be unable to get any information back:
>
> ![image-20201016000543310](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr1mv6a9vj30st0ghwgm.jpg)
>
> Our query does not know how to handle the company field, it does not know how to "resolve" this data. This is why we need a resolver for each an every query.



It's important to be aware of the data we're going to be querying for, so there's the structure of our Job data -- as we've already seen from our query.

```json
// data/jobs.json
{
  "id": "rJKAbDd_z",
  "companyId": "HJRa-DOuG",
  "title": "Frontend Developer",
  "description": "We are looking for a Frontend Developer familiar with React."
},
```

> **NOTE:** the job data also has a `companyId` field, which we can use to help us map our relation to the actual company it belongs to:

```json
// data/companies.json

{
  "id": "HJRa-DOuG",
  "name": "Facegle",
  "description": "We are a startup on a mission to disrupt social search engines. Think Facebook meet Google."
},
```

> It's important to note that different data may require different types of resolvers. In our case, our data current already has a reference to the company related to each job as a `companyid`, which matches the actual `id` of it's related company. We can use this as a `foreignKey` similar to what we find in relational databases. 



> **NOTE:** that our schema no dictates that the company field should resolve to a `Company` type, not just the `companyId` so this means our resolver will do the heavy-lifting to get and set the expected data:

```js
// resolvers.js

const Job = {
  // the job argument represents the all of our job data
  company: (job) => db.companies.get(job.companyId),
  // we use the job.companyId to query for the matching company, by it's id
};

module.exports = { Query, Job }; // be sure to export each of the resolvers
```

![image-20201016003004298](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr2c6zdvaj30t40h240s.jpg)



**The Parent Arugment (resolvers)**

`parent` => holds the reference to the parent type.

> graphql types maintain a heirarchy, which is what the nested structure of our queries represents. This also creates parent/child relationships between our data associations. In the example above the `parent` is the `job` type, so that is available to us as the first argument. Here we've logged out the contents of the `job` argument:
>
> ```shell
> {
>   id: 'rJKAbDd_z',
>   companyId: 'HJRa-DOuG',
>   title: 'Frontend Developer',
>   description: 'We are looking for a Frontend Developer familiar with React.'
> }
> {
>   id: 'SJRAZDu_z',
>   companyId: 'HJRa-DOuG',
>   title: 'Backend Developer',
>   description: 'We are looking for a Backend Developer familiar with Node.js and Express.'
> }
> {
>   id: 'rkz1GwOOM',
>   companyId: 'SJV0-wdOM',
>   title: 'Full-Stack Developer',
>   description: 'We are looking for a Full-Stack Developer familiar with Node.js, Express, and React.'
> }
> {
>   id: 'rJKAbDd_z',
>   companyId: 'HJRa-DOuG',
>   title: 'Frontend Developer',
>   description: 'We are looking for a Frontend Developer familiar with React.'
> }
> {
>   id: 'SJRAZDu_z',
>   companyId: 'HJRa-DOuG',
>   title: 'Backend Developer',
>   description: 'We are looking for a Backend Developer familiar with Node.js and Express.'
> }
> {
>   id: 'rkz1GwOOM',
>   companyId: 'SJV0-wdOM',
>   title: 'Full-Stack Developer',
>   description: 'We are looking for a Full-Stack Developer familiar with Node.js, Express, and React.'
> }
> ```
>
> It is simply all of our job data. This means we have access to the companyId, which we then use to look up the matching company using `db.companies.get(job.companyId)`



