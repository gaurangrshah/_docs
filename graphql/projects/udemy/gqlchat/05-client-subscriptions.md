---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Using Subscriptions in the Client
details:
  section: 6
  lesson: 40
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



One of the ways to decide how your application should handle your different queries, mutations and subscriptions is to think about how your application is currently consuming the data. What data is necessary to render which component? 

```js
// Chat.js

  async componentDidMount() {
    const messages = await getMessages();
    this.setState({ messages });
  }

  async handleSend(text) {
    const message = await addMessage(text);
    this.setState({ messages: this.state.messages.concat(message) });
  }
```

> In our case our chat component does most of the heavy lifting, including setting the current messages to state in the `componentDidMount()` method. This is also a good place to ensure our subscription has access to our new messages, 
>
> ------- ??? and we can even remove the `handleSend()` method as it becomes redundant.

```js
// Chat.js

// 🔵 get 'onMessageAdded' from queries
import { addMessage, getMessages, onMessageAdded } from "./graphql/queries";


  async componentDidMount() {
    const messages = await getMessages();
    this.setState({ messages });
    onMessageAdded((message) => {
      // updates state with any new messages
      this.setState({ messages: this.state.messages.concat(message) });
    });
  }

  async handleSend(text) {
    const message = await addMessage(text);
    // ❌  this.setState({ messages: this.state.messages.concat(message) });
  }
```

> Our `onMessageAdded` handler, has access to the consistent stream of our messages, when it new message is passed in, it updates the current state with a new message. 



Now that we have our client side subscriptions handling figured out, we'll need to update our queries to include our subscription:

```js
// graphql/queries.js


const messageAddedSubscription = gql`
  subscription {
    messageAdded {
      id
      from
      text
    }
  }
`;

```



And the `onMessageAdded()` 

```js
// graphql/queries.js

export async function onMessageAdded(handleMessage) {
  // subscribe to our message which returns an "observable"
  const observable = client.subscribe({ query: messageAddedSubscription });
  
  // take a look under the hood:
  console.log({ observable });
  observable.subscribe((result) => console.log({ result }));
}

```

> ![image-20201018032132911](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtij750utj30t40370sz.jpg)
>
> the observable only has that single function that we use to subscribe to the data
>
> ![image-20201018032036461](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtiia1sssj30so02xgls.jpg)
>
> That is the result includes a data object with our subscribed messages



So that we now what's on the data object we can work with it:

```js
// graphql/queries.js

export async function onMessageAdded(handleMessage) {
  // subscribe to our message which returns an "observable"
  const observable = client.subscribe({ query: messageAddedSubscription });
  return observable.subscribe(({ data }) => handleMessage(data.messageAdded));
}
```

> this keeps an active subscription to our messages array, as new messages come in they get added to this array, and we'll get the updates and pass them into `handleMessage()`, this will get our application to a working state, for starting subscriptions, but we have not yet handled stopping them. 



```js
// Chat.js

	
  subscription = null; // 🔵 declare a class property for our subscription

  async componentDidMount() {
    const messages = await getMessages();
    this.setState({ messages });
    // using the class property to keep a reference to the subscription
    this.subscription = onMessageAdded((message) => {
      // updates state with any new messages
      this.setState({ messages: this.state.messages.concat(message) });
    });
  }
```

> we've set a subscriptions property on the Class instance, which allows us to retain a reference to the subscription method `onMessageAdded()`, we can use this reference to cancel all subscriptions when the componenent unmounts:



```js
// Chat.js

componentWillUnmount() {
    // ❗️ make sure we clean up the subscription on unmoount
    if (this.subscription) {
      this.subscription.unSubscribe();
    }
  }
```

Now when our component unmounts it will automatically call the `unSubscribe()` method of our subscription since we've maintained a refernce to it as a property of our class instance. 

![chat-subscribed](https://tva1.sinaimg.cn/large/007S8ZIlly1gjtj5ozq8ng30kx0boasm.gif)

