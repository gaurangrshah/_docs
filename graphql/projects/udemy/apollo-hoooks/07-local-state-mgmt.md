---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Local State Management with Apollo Client
details:
  section: 7
  lesson: 50
---



## Local State Management with Apollo Client



In the last video we utilized react's own `useState` hook to help manage our application state for messages. Although apollo provides a way to manage local state as well, and we'll take a look at how we can implement state the apollo way.

> At its core, Apollo Client is a **state management** library that happens to use GraphQL to interact with a remote server. Naturally, some application state doesn't *require* a remote server because it's entirely local.
>
> Apollo Client enables you to manage local state alongside remotely fetched state, meaning you can interact with *all* of your application's state with a single API.
>
> [Managing local state](https://www.apollographql.com/docs/react/local-state/local-state-management/)



Apollo store already has a cache of all of our requests already made, this saves us back and forth trips to our server to make uneccessary calls. Apollo suggests that they'd like to be the `single source of truth` for all data in the client application. When using the apollo store in this way, we can no only store our results in the cache, but rather any data we need for any component in our application can use our apollo cache to manage it's own data. 



```js
// 🚧 refactor to use apollo-local-state
// ❌ const [messages, setMessages] = useState([]);
const { data } = useQuery(messagesQuery);
const messages = data ? data.messages : [];

useSubscription(messageAddedSubscription, {
  onSubscriptionData: ({ client, subscriptionData }) => {
    // ❌ setMessages(messages.concat(subscriptionData.data.messageAdded));
		
    // client will write messages to cache, for use with apollo-store
    client.writeData({
      data: {
        messages: messages.concat(subscriptionData.data.messageAdded),
      },
    });
  },
});
```



![Kapture 2020-10-18 at 19.27.44](https://tva1.sinaimg.cn/large/007S8ZIlly1gjuagt8mqog30u10u07wh.gif)