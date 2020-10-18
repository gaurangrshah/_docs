---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Enabling WebSockets in Apollo
details:
  section: 6
  lesson: 36
---



## Enabling WebSockets in Apollo



### What are websockets?

> The **WebSocket API** is an advanced technology that makes it possible to open a two-way interactive communication session between the user's browser and a server. With this API, you can send messages to a server and receive event-driven responses without having to poll the server for a reply. - [MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)



The type of two-way communication that websockets allow for fundamentally change the request/response nature of the normal http protocol. Websockets listen to events for changes and then react by updating the data accordingly based on the event. 



In order to implement websockets with apollo for our subscriptions, we'll need to update our server:

```js
// server.js

// import http to create a custom http subscription server
const http = require("http"); 

// create a new http server for our subscriptions
const httpServer = http.createServer(app); 

// geenerate apollo subscription handlers for our custom httpServer
apolloServer.installSubscriptionHandlers(httpServer);
// instead of listening with express, we're creating our own httpServer instance instead
// app.listen(port, () => console.log(`Server started on port ${port}`));
httpServer.listen(port, () => console.log(`Server started on port ${port}`));
```

> We've essentially replaced express's `app.listen()` with our own `httpServer` instance, this allows us to insert apollo-client's provided subscription handlers and will allow us to implement subscriptions.



Now if we run our query we'll get a completely different error: 

![image-20201018012954904](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtfb2kqydj311m07xaaz.jpg)

> This time around our websocket seems to be configured, but now we're getting an undefined async iterable error, which is simply because we have not created any resolvers to handle our subscription. 