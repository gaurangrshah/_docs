---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: GraphQL by Example
details:
  section: 2
  lesson: 10
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---

# [GraphQL Job Board](https://github.com/uptoskill/graphql-job-board)

[source]: (https://github.com/uptoskill/graphql-job-board)



Sample application used in the GraphQL by Example course.



Description:

Basic application with a minimal server that provides a simplistic front end to handle requests and basic authentication workflow. This will allow the project to focus more heavily on the GraphQL rather than the setting up of the server. 





## Dependencies



### Server Dependencies

```json
  "dependencies": {
    "body-parser": "1.19.0",
    "cors": "2.8.5",
    "express": "4.17.1",
    "express-jwt": "5.3.3",
    "jsonwebtoken": "8.5.1",
    "notarealdb": "0.2.2"
  },
  "devDependencies": {
    "nodemon": "2.0.4"
  }
```



### Client Dependencies

```json
  "dependencies": {
    "bulma": "0.9.0",
    "react": "16.13.1",
    "react-dom": "16.13.1",
    "react-router-dom": "5.2.0"
  },
  "devDependencies": {
    "react-scripts": "3.4.1"
  },
```



Install dependencies for both the server and the client

```shell
cd server && npm i
```

```shell
cd client && npm i
```





**Adding Apollo Server to an existing application**

```shell
npm i apollo-server-express graphql --save
```





```js
// server.js

const { ApolloServer, gql } = require("apollo-server-express");
```



We can now create a separate file from which we'll import our typeDefs and resovlers:

```graphql
# schema.graphql

type Query {
  greeting: String
}
```

```js
// resolvers.js

const Query = {
  greetng: () => "Hello World",
};

module.exports = { resolvers };
```



Create Apollo Server and we can then pass the entire app into apollo as middleware:

```js
//server.js

const apolloServer = new ApolloServer({typeDefs, resolvers});

// NOTE: 
apolloServer.applyMiddleware({app, path: "/graphql"}) 
// path value is set to "graphql" by default, we're listing it here for reference.
```



We will also need to pass our typeDefs and resolvers into our ApolloServer:

```js
// server.js
const fs = require('fs') // require the node fs module

// NOTE: we're calling `gql` as a function, not as a template literal: 

const typeDefs = gql(fs.readFileSync("./schema.graphql", {encoding: 'utf8'})) 
// read graphql file with readFileSync
// must also define the encoding typeas 'utf8' so that it is processed properly 
// and not as binary

const resolvers = require(readFileSync("./resolvers.js"))
```



With our server, typeDefs and resolver setup, we can test our query:

![image-20201015215107052](https://tva1.sinaimg.cn/large/007S8ZIlly1gjqxqs5eg3j30tk055jrq.jpg)





[**📋 DOCUMENTATION**](https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md)

1. [Create Job Type](01-create-job-type)

2. [Structure of GraphQL Query](02-query-structure.md)

3. [Associations](03-associations.md)

4. [Fetch Data](04-fetch-data.md)

5. [Job Details](05-job-details.md)

6. [Fetch Single Job](06-fetch-single-job.md)

7. [Handling Error Responses](07-error-responses.md)

8. [Fetch Single Company](08-fetch-single-company.md)

9. [Return Related Jobs](09-return-related-jobs.md)

10. [Create a Job](10-create-job.md)

11. [Mutation Best Practices](11-mutation-best-practice.md)

12. [Enforcing Authentication](12-enforce-auth.md)

    



