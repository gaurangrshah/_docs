---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Subscription Resolver with PubSub
details:
  section: 6
  lesson: 37
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



- [ ] install `graphql-subscriptions`

  ```shell
  npm i -S graphql-subscriptions
  ```







# Subscription Resolver with PubSub

We can start by editing our resolvers and using our `graphql-subscriptions` module to help update our resolvers.

```js
// resolvers.js

const { PubSub } = require("graphql-subscriptions"); // 🔵 import pubsub


const Subscription = {
  messageAdded: {
    subscribe: () => pubSub.asyncIterator("MESSAGE_ADDED"),
  },
};
```

> - the subscribe method returns an iterator (i.e., an object that can return multiple values
> - the iterator subscribes to the "MESSAGE_ADDED" event



With this in place you'll notice that we no longer have any errors in our playground. In fact, our status shows us that the client is now listening for updates. 

![pubsub-spinner](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtfz33qetg30lx0crwhz.gif)



### PubSub

> Subscriptions depend on use of a publish and subscribe primitive to generate the events that notify a subscription. `PubSub` is a factory that creates event generators that is provided by all supported packages. `PubSub` is an implementation of the `PubSubEngine` interface, which has been adopted by a variety of additional [event-generating backends](https://www.apollographql.com/docs/apollo-server/data/subscriptions/#pubsub-implementations). - [Subscriptions](https://www.apollographql.com/docs/apollo-server/data/subscriptions/)

> There are many different implementations of pubSub, and you could just as well roll your own, but there are several libraries that help implement a more robust pubSub strategy, which is better for appilications that scale. Our pubSub implementation is viable for a single server, and no more. 
>
> 
>
> Look at for a more robust option: [graphql-redis-subscriptions](https://github.com/davidyaha/graphql-redis-subscriptions)
>
> ```shell
> npm i -S graphql-redis-subscriptions
> ```
>
> the code for this is pretty much exactly what we have, we'd just need to use the redis `PubSub` class rather than the one from `graphql-subscriptions` like we currently are





So now that we have our client listening from the playground, we still need to implement the publish feature through our `addMessage()` resolver:



- firstly we'll be setting and listening for a specific message to be set back and forth, `MESSAGE_ADDED` , this string tells the client that a new message added event has just fired. This will update any clients current subscribed to this `MESSAGE_ADDED` event.

```js
// resolvers.js

// 🔵 set the subscription event
const MESSAGE_ADDED = "MESSAGE_ADDED";
// ☝️ this can be used when we publish messages and when we subscribe to them
```

> ❗️ We've pulled this out into it's own variable because we reuse it in two diffrrent places, and it makes sense to just reference the string with a variable. 



```js
// resolvers.js

const Mutation = {
  addMessage: (_root, { input }, { userId }) => {
    requireAuth(userId);
    const messageId = db.messages.create({ from: userId, text: input.text });
    // ❌ return db.messages.get(messageId);
    const message = db.messages.get(messageId);
    
    // trigger the pubSub 'MESSAGE_ADDED' event, include the new message object
    pubSub.publish(MESSAGE_ADDED, { messageAdded: message });
    
    return message; // return message -- as expected by our schema
  },
};
```

> - ☝️ instead of an immediate return we first publish the new message, using pubSub.
> - then we return the message back as expected and dictated by our schema. 
>
> The mutation still returns a message object, although now before we return it, we trigger a `MESSAGE_ADDED` event that informs any subscriptions of the new data, which can then be loaded in almost real-time. 



> **NOTE:** we're able to subscribe to our message added event from the playground, we see messages sent by all users as they go across our server, but the users still cannot see each others messages without refreshing their browsers. 



So the next step is to implement the client-side apollo subscription in our application. 