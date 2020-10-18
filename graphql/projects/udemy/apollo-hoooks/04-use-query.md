---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: The useQuery Hook
details:
  section: 7
  lesson: 47
---



# Use Query Hook



```js
// Chat.js

import { useQuery } from "@apollo/react-hooks";


const Chat = ({ user }) => {
  const result = useQuery(messagesQuery);
  console.log({ result });

  /*...*/
};
```

> We'll notice that our log actually gets printed twice, the first time represents the first call, which is still trying to get the data. The next call actually contains the result that we are expecting: 
>
> ![image-20201018171242948](https://tva1.sinaimg.cn/large/007S8ZIlly1gju6k1oflxj30tk087myr.jpg)
>
> The next log statement is the log statement from after our query has completed successfully and it does contain our messages:
>
> ![image-20201018171326125](https://tva1.sinaimg.cn/large/007S8ZIlly1gju6krsnkxj30to08lmyx.jpg)
>
> Under the hood the `useQuery` hook fetches our data, handles our loading and error states, we just get to decide what we want to do with that data and/or errors afterwards



Now that we have a better idea of what the result from `useQuery` returns, we can destructure the items we need off of it:

```js
const { loading, error, data } = useQuery(messagesQuery);
```



Also note, we can pass our apollo client options into the useQuery hook as a second argument:

```js
const { data } = useQuery(messagesQuery, {
  // optional apollo-client config properties:
  variables: variables, 
  fetchPolicy: 'no-cache'
});
```



