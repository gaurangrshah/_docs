---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Enforce Authentication with context
details:
  section: 4
  lesson: 24
docs: https://github.com/gaurangrshah/_docs/blob/graphql/graphql/projects/udemy/graphql-job-board/setup.md
---



- [ ] [expressJWT](https://github.com/auth0/express-jwt) - handles jwt verification.





## Enforcing Authentication with Context

Currently when creating jobs from the client-side we've actually hard-coded a companyId, so all of our jobs are currently being created for a single company. What we need to really do is to be able to get the company associated with the logged in user, and be able to set the id from the context of the logged in user. 

In order to do this we'll need to leverage third argument in a graphql resolver which is the `context` and implement some basic authentication.

Another consideration is that anyone can create a new job currently whether or not they're logged in. 

![why-auth](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs68vyu8ig30to0q6qv8.gif)

> We are hiding the Post New Job link on the client side, but that isn't enough, someone who knows they can create a new post by going to the new job post url. Nothing is stopping them from doing so. We are not checking to see if the user who is submitting this is logged in or to even see if they belong to a company for that matter. 



Finally we can also create jobs from our graphql editor without authentication as well, which should not be the case. Luckily we can implement authentication across the board and protect both the client side app as well as the server side api. 



Currently we do have some authentication logic, so we're not starting from scratch. All we're doing is mocking the login process in our `server.js` file.

Lets start by talking a look at where the current login logic is implemented:

```js
// server.js

app.post("/login", (req, res) => {
  const { email, password } = req.body;
  const user = db.users.list().find((user) => user.email === email);
  if (!(user && user.password === password)) {
    res.sendStatus(401);
    return;
  }
  const token = jwt.sign({ sub: user.id }, jwtSecret);
  res.send({ token });
});
```

> - the `/login` route passes the email and password from the form into the request.body
>
> - we use the email to find the matching user and literally compare their actual password "string"  
>
>   ❗️**NOTE:** there is no hashing encryption logic implemented. 
>
> - if the passwords match we attempt to sign a token and return it back with the response
>
> - if the request fails, (i.e. user not found, token verification fails, or passwords didn't match, then we send a `401` error)



**Verification**

```js
// server.js

app.use(
  cors(),
  bodyParser.json(),
  expressJwt({
    secret: jwtSecret,
    credentialsRequired: false,
  })
);
```

> The app is configured to us a jsonWebToken although configured by default the `credentialsRequired` attribute to false currently. This is done through the [`expressJWT`](https://github.com/auth0/express-jwt) package



### Using Context

configure the `context` property on `apollo-server`

```js
// server.js

const apolloServer = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // adding the context property and providing access to our express request:
    method: req.method; // set the method property from the request to context
  },
});

```

> context is simply an object that apollo-server maintains for us, it makes it a great way to give our resolvers access to some of our core functionality such as authentication and even response and request handling or also error handling. 
>
> In the above example the context is set to a function that has access to our express request object, to make this clearer let's refactor this just a bit:
>
> ```js
> // server.js
> 
> const context = ({ req }) => ({
>   // setting context from the express request object:
>   user: req.user,
>   method: req.method,
> });
> 
> 
> const apolloServer = new ApolloServer({
>   typeDefs,
>   resolvers,
>   context, // setting context with apollo-server
> });
> ```
>
> This wires up our server giving Apollo-server direct access to the context we've set using express's request object.
>
> - currently we're setting the user property on our context, so now we can handle some authentication.



This `context` is now accessible as the third parameter from any graphql resolver function thanks to our `apollo-server`. Let's update our resolvers to access the user details from the context parameter:

```js
// resolvers.js

	// access the context param from our resolver:
  createJob: (root, { input }, context) => {
    console.log(context) // log out the context object !(to our node console)
    const id = db.jobs.create(input);
    const job = db.jobs.get(id);
    return job;
  },
```

> 🚧 currently when we run this in our playground we'll notice that our method gets logged from our request, but the user remains undefined. This is because we are not authenticating our user from playground so there is no identity to be able to set. 

![user-fail](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs98jtvt8g30to0p4x6r.gif)

> **Compare with the Front End Behavior**
>
> If we take a look at the front end login process, the response that is received contains our user's verified token, that token only gets set if the user is successfully authenticated through 

![login-token-set](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs8u0lqvvg30to0q6b2k.gif)

> 💡We can now use this token to send back to the server to make authenticated requests to prove the user is who they say they are and that they do have access to this endpoint and/or functionality. 

In fact let's copy and paste the token from the front end request to make requests from our graphql playground and we'll notice that we do in fact get the user details logged to our console:

![user-auth-playground](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs9e9k14kg30to0p4npk.gif)



So now that we have access to the logged in user, we can go ahead and secure our endpoint so only logged in users can make a createJob request. 

```js
// resolvers.js

  createJob: (root, { input }, { user }) => {
    // check if user is authenticated before allowing action
    if (!user) throw new Error("Unauthorized");
    // if user is authenticated then create job:
    const id = db.jobs.create(input); // createJobInput
    // use jobId to query for the job data
    const job = db.jobs.get(id);
    return job;
  },
```



Now let's test our authentication:

![succes-auth](https://tva1.sinaimg.cn/large/007S8ZIlly1gjs9so97krg30to0p4qvj.gif)