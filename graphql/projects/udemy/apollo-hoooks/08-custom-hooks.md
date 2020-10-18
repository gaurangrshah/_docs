---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Custom React Hooks
details:
  section: 7
  lesson: 51
---



## Custom React Hooks



Hooks allow us to write and extract, reusable logic for react components so that it may be directly used in other components. To get an idea of how they are best used, we'll refactor our messages to use a `useChatMessages` hook that with encompass all of our logic.



```js
// hooks.js


import { useQuery, useMutation, useSubscription } from "@apollo/react-hooks";
import {
  messagesQuery,
  addMessageMutation,
  messageAddedSubscription,
} from "./graphql/queries";


export function useChatMessages() {
  const { data } = useQuery(messagesQuery);
  const messages = data ? data.messages : [];
  console.log({ messages });
  useSubscription(messageAddedSubscription, {
    onSubscriptionData: ({ client, subscriptionData }) => {
      console.log({ subscriptionData });
      client.writeData({
        data: {
          messages: messages.concat(subscriptionData.data.messageAdded),
        },
      });
    },
  });

  const [addMessage] = useMutation(addMessageMutation);

  return {
    messages,
    addMessage: (text) => addMessage({ variables: { input: { text } } }),
  };
}
```

> we've encompassed all of our stateful logic into a hook that we can then borrow from to enable messages in our application:



```js
// Chat.js

import React from "react";
import { useChatMessages } from "./hooks";
import MessageInput from "./MessageInput";
import MessageList from "./MessageList";

const Chat = ({ user }) => {
  const { messages, addMessage } = useChatMessages();
	// 🚧 removed in custom hook refactor
  // ❌ const handleSend = async (text) => {
  //   // ❌  await addMessage({ variables: { input: { text } } });
  //   await addMessage(text);
  // };

  return (
    <section className='section'>
      <div className='container'>
        <h1 className='title'>Chatting as {user}</h1>
        <MessageList user={user} messages={messages} />
        {/* <MessageInput onSend={handleSend} /> */}
        <MessageInput onSend={addMessage} />
        {/* the new hook allows us to just expose the addMessage function */}
      </div>
    </section>
  );
};

export default Chat;
```





