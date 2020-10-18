---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Handling GraphQl Error Responses
details:
  section: 2
  lesson: 18
---



## Handling Error Responses

By default the errors provided in our client-side application, will not be very informative when it comes to our requests. Here's an example of a query that has a mis-spelled title field in its query. This means we'll receive an error, but as we'll notice it will not give us any clue to what the issue really is:

```js
export async function loadJobs() {
  const jobsQuery = `query JobsQuery{
				jobs {
					id
					titlez
					company {
						id
						name
					}
				}
			}`;
  const data = await graphqlRequest(jobsQuery);
  return data.jobs;
}
```

![image-20201016120651165](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrmh6tbufj30xj0gy42x.jpg)

> The error handler tells us that our error is due to it being unable to evaluate `data.jobs`, which is technically true, because there is not data.jobs due to the fact that we mis-spelled the  field `title` our query. But we wouldn't know this from this error, we'd have to dig a bit further into the request itself for the actual problem:
>
> ![image-20201016121003255](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrmkii6y6j30tr0fw771.jpg)
>
> The error returned in our response from the server contains our actual issue





Now that we understand how errors work with graphql requests, let's make sure that we're handling them properly on the client side so that we can display the more informative error messages we get back from our graphql server in our `graphqlRequest()`:

```js
// requests.js

export async function graphqlRequest(query, variables = {}) {
  // @param: query = graphql query * required
  // @param: variables = graphql varaibale arguments (optional)
  const response = await fetch(devEndpointURL, {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ query, variables }),
  });
  const responseBody = await response.json();
  
  if (responseBody.errors) {
    const message = repsonseBody.errors
   // errors is an array, and can contain multiple error messages
      .map((error) => error.message)
   // we want to map thru each and return them as a string separated by a line break
      .join("\n");
    throw new Error(message);
  }
  
  return responseBody.data; // return the root data object directly
}
```

> And now our backend graphql errors are displayed to the client side:
>
> ![image-20201016122325454](https://tva1.sinaimg.cn/large/007S8ZIlly1gjrmygkrlhj30xe0u0jxy.jpg)