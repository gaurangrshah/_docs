---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: The useSubscription Hook
details:
  section: 7
  lesson: 49
---



## The useSubscription Hook

The useSubscription hook actually makes many of the complicated aspects of using subscriptions easier. 

```js
  const { data } = useQuery(messagesQuery);

  useSubscription(messageAddedSubscription, {
    // since we want to act upon the data we get from the subscription, we're
    onSubscriptionData: (result) => {
      console.log(result);
    },
  });
```

> **❗️NOTE: ** `useSubscription` cannot be called before the query it is being subscribed to. 
>
> - there are several helpers available from useSubscription that we can use in other use cases, i.e.:
>
>   ```js
>   const {data, error, loading } = useSubscription(messageAddedSubscription)
>   ```
>
> -  -- in our case we simple want to act upon the data immediately, so we're opting for asynchonrous syntax 



Below we checking the logged statement from the code above.

![image-20201018185240771](https://tva1.sinaimg.cn/large/007S8ZIlly1gju9g1q0wwj30tr05gq46.jpg)

Here we see that `subscriptionData` is the result of our subscription query, it contains a data property with our messageAdded object appended to it. In order for our messages to get added back into our app when we create new messages, well need to ensure we're returning the data property itself, lets double check and make sure we're targeting the right data:

```js
// Chat.js

  const { data } = useQuery(messagesQuery);
  useSubscription(messageAddedSubscription, {
    onSubscriptionData: ({ subscriptionData }) => {
      console.log("onSubscriptionData", subscriptionData.data.messageAdded);
    },
  });
```

![image-20201018185550261](https://tva1.sinaimg.cn/large/007S8ZIlly1gju9jc0bf0j30tn03wmxo.jpg)

Now we're getting exactly the data that we need to from our subscriptionData



In order to properly handle messages, we'll need to create a big of state to hold each new message in our component as new messages get added

```js
// Chat.js

// 🔵 instantiate state to hold all of our messages
const [messages, setMessages] = useState([]);
```



Then we'll need to update our query to make sure it is able to save our messages to our state, and we can do this similar to how we handle new messages with `useSubscription`

```js
// Chat.js

  const [messages, setMessages] = useState([]);

  // ❌ const { data } = useQuery(messagesQuery);
  useQuery(messagesQuery, {
    // handler to update state with after query response
    onCompleted: ({ messages }) => setMessages(messages),
  });

```

> because we're working with data from two different hooks, `useQuery` and `useSubscription`, we needed some piece of local state to help take the results from both and ensure that they get added to our application state accurately. 





Lastly we can update our `useSubscription` hook as well, to incorporate our new messages state.

```js
// Chat.js

  useSubscription(messageAddedSubscription, {
    onSubscriptionData: ({ subscriptionData }) => {
      setMessages(messages.concat(subscriptionData.data.messageAdded));
    },
  });
```

> **NOTE: ** using apollo's hooks allows us to greatly simply how our subscription logic works when compared to using apollo-client directly. We no longer have to worry about cleaning up the subscription as apollo handles that for us automatically when using the hooks implementation. 