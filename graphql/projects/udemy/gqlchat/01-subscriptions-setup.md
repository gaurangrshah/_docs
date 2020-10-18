---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Chat Application Setup
details:
  section: 6
  lesson: 33
---



# [GraphQL Chat](https://github.com/uptoskill/graphql-chat)

[source]: (https://github.com/uptoskill/graphql-chat)	"GraphQL Job Board Repo"



Sample application used in the GraphQL by Example course.



## Dependencies



### Server Dependencies

```json
  "dependencies": {
    "apollo-server-express": "2.14.3",
    "body-parser": "1.19.0",
    "cors": "2.8.5",
    "express": "4.17.1",
    "express-jwt": "5.3.3",
    "graphql": "14.6.0",
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
    "apollo-boost": "0.4.9",
    "bulma": "0.9.0",
    "graphql": "14.6.0",
    "jwt-decode": "2.2.0",
    "react": "16.13.1",
    "react-dom": "16.13.1"
  },
  "devDependencies": {
    "react-scripts": "^3.4.3"
  },
```



Install dependencies for both the server and the client

```shell
cd server && npm i
```

```shell
cd client && npm i
```





So currently our app has working authentication, with a very simply api, and a semi-working chat application. Our application has one specific problem when users message each other, their messages won't show in the other user's chat window unless that user actually refreshes their page:

![chat-refresh](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtdyo2fl4g30qx0cunkz.gif)



### Subscriptions

> Subscriptions are GraphQL operations that watch events emitted from Apollo Server.  - [Subscriptions](https://www.apollographql.com/docs/apollo-server/data/subscriptions/)



In order to fix this issue we can use graphql subscriptions are another query type just like mutations and queries are. Subscriptions will allow us to monitor our messages data for changes so we can think of them as a subscription to the querys and mutations associated with our specific data that we subcribe to:

```
# schema.graphql

type Subscription {
  messageAdded: Message
}
```

> Subscriptions generally subscribe to events related to the data itself. Here we're not actually creating a subscription to the `Message` type, but instead to the event that occurs when a message is actually added. From that event we the expect a `Message` object

> **☝️TIP:** Since we only updated the schema, nodemon will not pick up the changes we've made, so we can restart nodemon manually simply by typing `rs` in the console. We can then verify the server has received our schema update, by checking the schema in our playground:
>
> ![image-20201018005312285](https://tva1.sinaimg.cn/large/007S8ZIlly1gjte8vj3khj311q0lrgnp.jpg)



Now if we try to run our subscription, we'll see a new error:

![image-20201018005802454](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtedw4otwj311y0c2abm.jpg)

<iframe width="600" height="400" src="http://localhost:9000/graphql" marginwidth="0" marginheight="0" frameborder="no" scrolling="yes"></iframe> 

> **Could not connect to websocket endpoint**
>
> The first thing to note is that the endpoint listed in the error is using a completely different protocol, if you'll notice in our playground we're actually subscribing to our normal `http` graphql endpoint at port: 900, but our error is trying to us a `ws` protocol. Which is a `web socket`  -- "this is because `subscriptions` are usually implemented with `WebSockets`, where the server holds a steady connection to the client".  - [Realtime Updates with GraphQL Subscriptions](https://www.howtographql.com/react-apollo/8-subscriptions/)



