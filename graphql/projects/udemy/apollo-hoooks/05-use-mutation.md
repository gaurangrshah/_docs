---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: The useMutation Hook
details:
  section: 7
  lesson: 48
---





## The useMutation Hook

Unline the useQuery hook, the useMutationHook does not trigger the mutation automatically, instead it returns a setter function that we can the use to trigger the mutation manually. The variable we get back can be named anything we want so it makes sense to name it after the mutation we want to support:

```js
// Chat.js

// const [mutate] = useMutation(addMessageMutation);
const [addMessage] = useMutation(addMessageMutation);

```



Now that we have the updater function for our mutation, we can then use it to update our messages in our `handleSend()` function.

```js
// Chat.js

const handleSend = async (text) => {
  const { data } = await addMessage({ variables: { input: { text } } });
  console.log(data);
};
```

![Kapture 2020-10-18 at 18.32.58](/private/var/folders/0p/llym455n4h715mkmrnsjf5fc0000gn/T/203226db68596695851c11602275e118/Kapture 2020-10-18 at 18.32.58.gif)

As we can see we are able to get the message in the console, although it still will not appear in our chat windows without a refresh yet. We'll need to update our subscription handler in order ensure that the new messages can update any subscriptions as well. 

```js
// Chat.js

const [
  addMessage,
  {
    // optoinal second object from useMutation
    loading, // loading state indicator
    error, // error state
    data, // current data object
    called, // status of mutation (mutationResult)
  },
] = useMutation(addMessageMutation);
```

> Before we go any further it's important to know that `useMutation` also has some of the same helpers as we got form `useQuery`, the biggest difference is that we also get a `called` property available to us from `useMutation` that holds the current status of our mutation -- whether or not it has been called. 



