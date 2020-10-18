---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Setting up Apollo Provider
details:
  section: 7
  lesson: 45
---



- [x] install @apollo/react-hooks

  ```shell
  npm i -S @apollo/react-hooks
  ```

  

## Setting up Apollo Provider



We can setup the Apollo-provider at a top-level of our application in our case that will be the `App.js` file. We'll need to wrap our entire application with the apollo provider:

```js
// App.js

import {ApolloProvider} from "@apollo/react-hooks"

class App extends Component {
  state = { user: getLoggedInUser() };

  handleLogin(user) {
    this.setState({ user });
  }

  handleLogout() {
    logout();
    this.setState({ user: null });
  }

  render() {
    const { user } = this.state;
    if (!user) {
      return <Login onLogin={this.handleLogin.bind(this)} />;
    }
    return (
      
      <ApolloProvider>
        <NavBar onLogout={this.handleLogout.bind(this)} />
        <Chat user={user} />
      </ApolloProvider>

    );
  }
}
```



Apollo Provider requires a single argument, that sets up the client instance for our apollo-provider:

```js
// App.js

import client from "./graphql/client";

    return (
      /*  pass client into apollo provider */
      <ApolloProvider client={client}>
        {/*  wrap application with apolloprovider */}
        <NavBar onLogout={this.handleLogout.bind(this)} />
        <Chat user={user} />
      </ApolloProvider>
    );
```

> This gives our entire app access to the apollo-client instance