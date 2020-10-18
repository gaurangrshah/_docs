---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Configuring WebSocketLink in Apollo Cient
details:
  section: 6
  lesson: 39
---





## Configuring WebSocketLink in Apollo Cient



So far we have our subscriptions working in our playground, but we'll need to update our client side application in order to implement subscriptions on the front end, the first step in doing so is adding support for WebSockets in our client side app. 

```shell
npm i -S apollo-link-ws subscriptions-transport-ws
```

> - [apollo-link-ws](https://www.apollographql.com/docs/link/links/ws/): Send GraphQL operations over a WebSocket. Works with GraphQL Subscriptions.
> - [subscriptions-transport-ws](https://github.com/apollographql/subscriptions-transport-ws): A GraphQL WebSocket server and client to facilitate GraphQL queries, mutations and subscriptions over WebSocket.



**Create new Web Socket Instance**

```js
// graphql/client.js

// 🔵 import WebSocketLink to send operations over websockets
import { WebSocketLink } from "apollo-link-ws";

const wsUrl = "ws://localhost:9000/graphql"; // 🔵 websocket protocol uri

const wsLink = new WebSocketLink({
  uri: wsUrl,
  options: {
    // false by default, true === deferred until a websocket connection is requested
    lazy: true,
    reconnect: true, // if the websocket connection is interrupted, the client will try to reconnect
  },
});
```



Next we'll need to configure apollo-client to user our `wsLink` for subscriptions, but we still want to make sure that we're able to make our original `http` requests as well, so we'll have to incorporate them both with `split` 

```js
// graphql/client.js

import {
  ApolloClient,
  ApolloLink,
  HttpLink,
  InMemoryCache,
  split,
  // 🔵 import split to help split is a conditional wrapper that displays ApolloLink for http requests and can then show our WebSocketLink for ws protocol requests
} from "apollo-boost";
```

> `split` is a conditional wrapper that displays `ApolloLink` for http requests and can then show our `WebSocketLink` for ws protocol requests, we can use it to update our client:
>
> ```js
> // graphql/client.js
> 
> const client = new ApolloClient({
>   cache: new InMemoryCache(),
>   link: httpLink,// we'll want to conditionally use httpLink and WebSocketLink
>   defaultOptions: { query: { fetchPolicy: "no-cache" } },
> });
> ```



First we'll need to check if there is an active subscription, this will tell us which logic we need to implement for whatever protocol is being requested:

```js
// graphql/client.js
function isSubscription(operation) {
  // getMainDefinition to extract the definition object from the query:
  const definition = getMainDefinition(operation.query);

  return (
    definition.kind === "OperationDefinition" &&
    definition.operation === "subscription"
  );
}
```

> we get into some really low level apollo stuff to operate on the query object itself,  but essentially we are just verifying if our definition matchins some of the presets of a subscription, if so we know we currently have an acive subscription.



```js
// graphql/client.js

const client = new ApolloClient({
  cache: new InMemoryCache(),
  // link: httpLink,
  link: split(isSubscription, wsLink, httpLink), 
  // =>  if isSubscription ? wsLink : httpLink
  defaultOptions: { query: { fetchPolicy: "no-cache" } },
});
```

> split is simply a conditional wrapper that takes a condition and when true will display 



This should handle the subscription for us from the client side application, but we still need to configure our client to update when a new message is added, we can listen to websockets, but we must now react when changes occur. 