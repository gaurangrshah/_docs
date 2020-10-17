---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Extracting the Company from the Authenticated user
details:
  section: 4
  lesson: 26
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



## Extract Data from Authenticated user

Our current data allows access to company information from the logged in user via the companyId property on each user. When a user logs in their id is recieved by the server as the `sub` property. this is the id we set the context with as well. Let's take a look at how we can get the company information with the user id. 

In the end we'll want to be able to infer the each company that a job is created for, by the user's association with that company. This way we can associate the job to both the user and their company. 

```js
// server.js

🚧 refactor //  const context = ({ req }) => ({
            //   // setting context from the express request object:
            //   user: req.user,
            //   method: req.method,
            // });

// grab user object from database using the id stored as "sub" = "subject"
const context = ({ req }) => ({ user: req.user && db.users.get(req.user.sub) });
// as a fail-safe we only set the user if the user id exists.
```

> in this refactor instead of passing the specific properties that we get set from our token, we opt to query the database to find the matching user, and return the entire user, setting the entire object to our apollo context. 



```js
// resolvers.js

    // 🚧 refactor // const id = db.jobs.create(input); 
    
		// using the authenticated user to set the companyId
    const id = db.jobs.create({ ...input, companyId: user.companyId });
    
		const job = db.jobs.get(id); // use jobId to query for the job data
    return job;
  },
```



Lastly, we no long require the companyId be set in our schema as we're handling it's logic under the hood manually and no longer-hard coding it. 

```
# schema.graphql

input CreateJobInput {
  # companyId: ID! # remove companyId field
  title: String
  description: String
}
```



So now we can try our mutation without passing in any of the companyId in the variable and our request should still succeed:

![image-20201017022227677](/Users/bunty/Library/Application Support/typora-user-images/image-20201017022227677.png)





We can now also remove the hard-coded companyId implementation from the front-end as well:

```js
// JobForm.js

// const companyId = "HJRa-DOuG"; //🚧  refactor : FIXME: temporarily hard-coding the companyId:
    // const { title, description } = this.state;
    // createJob({ companyId, title, description }).then((job) => {

	createJob(this.state).then((job) => {
      this.props.history.push(`/jobs/${job.id}`);
    });
```

> we no longer need to pass the hard-coded companyId, so we've simplified the logic to aslo just pass in the entire state object rather that destructuring each property.
>
> And this should also succeed from the front end:

