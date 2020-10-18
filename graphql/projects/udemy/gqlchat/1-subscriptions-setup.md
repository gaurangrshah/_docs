---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Chat Application Setup
details:
  section: 6
  lesson: 33
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---

# [GraphQL Job Board](https://github.com/uptoskill/graphql-chat)

[source]: (https://github.com/uptoskill/graphql-chat)	"GraphQL Job Board Repo"



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



