---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Client Authentication with WebSockets
details:
  section: 6
  lesson: 42
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



Now that we have our websockets configured and our application secured, we'll need to do the same for our websocket connection from our playground, at the moment anyone including unauthenticated users can manipulate our data from the playground, we need to ensure that we restrict our access by implementing authentication on the client side.



The way we'll implement authentication for our websocket connections will be a bit different, but we'll be taking advantage of the payload property given to us by the websocket initiation request: 

![Kapture 2020-10-18 at 12.24.14](https://tva1.sinaimg.cn/large/007S8ZIlly1gjty94g6x2g30m80cs1fu.gif)

As we can see the websocket also receives our messages through the same payload type of data structure as our initial http request, we'll be using this same object to help us send our authentication credentials back and forth between our client and server.



This can be achieved by using the `connectionParams` property provided on our WebSocketLink

```js
// graphql/client.js

const wsLink = new WebSocketLink({
  uri: wsUrl,
  options: {
    //  ☝️  connectionParams help us add details to our websocket connectionParams
    connectionParams: {
      accessToken: getAccessToken(),
      // in our case we'll be setting our auth credentials here.
      // similar to how we store them in our headers for http requests
    },
    lazy: true,
    reconnect: true,
  },
});
```



With this in place we're now sending along our accessToken along with our subscription request:

![image-20201018123553991](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtyk0vewmj317f0txjwr.jpg)

The issue so far is that we're only setting the token if the user is already logged in. We'll need to handle a similiar configuration for the use case of when a new user logs in, we'll also want to initiate the request with the token in the payload as well. 

Because our token currently gets set if and when the applicatin first starts, we may not have a user logged in already at this point, so if a user logs in after this point then we never actually end up setting the token again, the value for the token will remain null. In order to fix this we can modify our code slightly:

```js
// graphql/client.js

const wsLink = new WebSocketLink({
  uri: wsUrl,
  options: {

    connectionParams: () => ({
      // ☝️ make connectionParams a fn() to ensure the token is set each time a connection is initiated
      accessToken: getAccessToken(),
      // in our case we'll be setting our auth credentials here.
      // similar to how we store them in our headers for http requests
    }),
    lazy: true,
    reconnect: true,
  },
});
```

> turning our connectionParams configuration into a function that returns the token will ensure that each time a connection is started the token will get set if it exists. 



We can test this to ensure that a token gets set each time we create a new connection even if we're logging 

![Kapture 2020-10-18 at 14.33.33](https://tva1.sinaimg.cn/large/007S8ZIlly1gju1yt96o6g30m80cs1fu.gif)