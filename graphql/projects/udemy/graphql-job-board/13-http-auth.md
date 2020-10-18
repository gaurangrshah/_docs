---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Passing Authentication in HTTP Requests
details:
  section: 4
  lesson: 25
---



## Passing Authentication in HTTP Requests

Now that we've secured our graphql createJobs mutation, we can no longer create a job on the front end client side application. 

![image-20201017013810188](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs9xbydhqj30tk0cbgnr.jpg)



> We get our `Unauthorized` error that we threw, meaning we don't have access to a user identity. This is because we'll need to set our authorization headers when making our front-end request as well



In order to achieve this we'll need to refactor our `graphqlRequest()`

```js
// requests.js

// auth utils
import { isLoggedIn, getAccessToken } from "./auth"; 


  const request = {
    // extracted request into a separate object
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ query, variables }),
  };

  // add auth headers to request if user is authenticated:
  if (isLoggedIn()) {
    request.headers["authorization"] = "Bearer" + getAccessToken();
  }
  // pass in the new request object if user is loggedIn -- will include headers:
  const response = await fetch(devEndpointURL, request);
```

> ❗️**NOTE:** we've implemented logic that uses two functions to check if the user is logged in and get the current access token of the logged in user in order to set the request headers.
>
> these two functions already exist for us:
>
> ```js
> // auth.js
> 
> const accessTokenKey = 'accessToken';
> 
> export function getAccessToken() {
>   return localStorage.getItem(accessTokenKey);
> }
> 
> export function isLoggedIn() {
>   return !!localStorage.getItem(accessTokenKey);
> }
> ```
>
> > When the user logs in we're setting the current token to local storage, when we need to authenticate, we're then grabbing that token from local storage and passing it along with our request headers IF our user is authenticated, if not we throw an `Unauthorized` error. 
>
> 
>
> ❗️❗️❗️❗️**NOTE:** NOT FOR PRODUCTION: the file aslo has a note :
>
> ```js
> // NOTE: this example keeps the access token in LocalStorage just because it's simpler
> // but in a real application you may want to use cookies instead for better security
> ```
>
> 

So here's our auth workng:

![final auth](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsz6mtt30g30to0rqx6s.gif)

