---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Intro to React Hooks
details:
  section: 7
  lesson: 46
---

-----

**REMOVABLE**

Currently our app component is composed as a `Class` based component, in order to use the hooks that `apollo-hooks` gives us, we'll need to convert this component into a functional component. 

```js
// App.js

const App = () => {
  const [state, setState] = useState({ user: getLoggedInUser() });

  const handleLogin = (user) => setState({ user });

  const handleLogout = () => {
    logout();
    setState({ user: null });
  };

  return state.user ? (
    /* pass client into apollo provider */
    <ApolloProvider client={client}>
      {/*wrap application with apolloprovider */}
      <NavBar onLogout={handleLogout} />
      <Chat user={state.user} />
    </ApolloProvider>
  ) : (
    <Login onLogin={handleLogin} />
  );
};

export default App;
```

**REMOVEABLE**

----





Currently our Chat component is composed as a `Class` based component, in order to use the hooks that `apollo-hooks` gives us, we'll need to convert this component into a functional component. 

```js
// Chat.js

class Chat extends Component {
  state = { messages: [] };
  subscription = null; // 🔵 declare a class property for our subscription

  async componentDidMount() {
    const messages = await getMessages();
    // 🚧  subscriptions refactor
    this.setState({ messages });
    // using the class property to keep a reference to the subscription
    this.subscription = onMessageAdded((message) => {
      // 🔵  updates state with any new messages
      this.setState({ messages: this.state.messages.concat(message) });
    });
  }

  componentWillUnmount() {
    // ❗️ make sure we clean up the subscription on unmoount
    if (this.subscription) {
      this.subscription.unSubscribe();
    }
  }

  async handleSend(text) {
    const message = await addMessage(text);
    // ❌  this.setState({ messages: this.state.messages.concat(message) });
  }

  render() {
    const { user } = this.props;
    const { messages } = this.state;
    return (
      <section className='section'>
        <div className='container'>
          <h1 className='title'>Chatting as {user}</h1>
          <MessageList user={user} messages={messages} />
          <MessageInput onSend={this.handleSend.bind(this)} />
        </div>
      </section>
    );
  }
}

export default Chat;

```

> that's the original, now let's create our functional counterpart:

```js
const Chat = ({ user }) => {
  const [messages, setMessages] = useState([]);

  const handleSend = (text) => {
    // 🔵  create a dummy message to test our state logic
    const message = { id: 1, text, from: "you", text };
    setMessages(messsages.concat(message));
  };

  return (
    <section className='section'>
      <div className='container'>
        <h1 className='title'>Chatting as {user}</h1>
        <MessageList user={user} messages={messages} />
        <MessageInput onSend={handleSend} />
      </div>
    </section>
  );
};

export default Chat;
```

> For now you'll notice we haven't setup all of our functionality, instead we've opted to just get our app back to a working state, while we update the rest of the functionality



