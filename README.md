🚀 Live Site: https://gun-chat-zeta.vercel.app/

---

## What is GUN

> GUN.js a decentralized graph database for freedom fighters.

Graph database applications have grown dramatically during the last few years. Graph databases, for example, are already utilized by Facebook for their social media platform, Stripe for fraudulent transactions, Amazon for product recommendation, and companies all over the world for big data analytics in a variety of sectors and challenges.

Unlike a centralized database that stays on a server maintained by big tech, data in GUN is dispersed between several peers or users using the power of WebRTC. This decentralized database functions exactly like a cloud database from the developer’s perspective; however, it is hosted on a completely peer to peer network. It is not a blockchain, but leverages some of the same cryptographic algorithms, such as Patricia-Merkle trees.

In this post, we'll utilize GUN to build a decentralized chat software that employs user authentication and end-to-end encryption.

---

## Screenshots

![Login Page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1al4rbdjemh9t5toepi0.png)

![Chat Section](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dd4hky6qqrgv4jkskos5.png)

---

## Step 1 - Setting up the Environment

We will use slimline as our frontend framework, so create a new project directory called '`chat-app`' and run command to initialize npm.

```bash
npm init -y
```

to create a svelte app run

```bash
npm create vite@latest chat-app --template svelte
```

Following that, we will require a gun for storage and to install run

```
npm install gun
```

---

## Step 2 - Implementing User Authentication

Create a new file called `user.js` in the project's `src` subdirectory, where we will implement our user authentication.

Following that, we will import Gun as well as the side libraries known as `sea` and `axe`. sea represents security, encryption, and authorization, and it allows for user authentication. axe stands for advanced exchange equation, and it is a different technique to link peers.

```js
import GUN from "gun";
import "gun/sea";
import "gun/axe";
```

Following that, we will initialize our database and make a database reference to the currently authenticated user.

```js
// Database
export const db = GUN();

// Gun User
export const user = db.user().recall({ sessionStorage: true });
```

We used `sessionStorage: true` to prevent the user from being logged out while closing and reopening tabs.

Next, we'll need the username, which will be used frequently in the app, so we'll import writable from svelte/store at the start of the file to make the app more responsive.

```js
import { writable } from "svelte/store";
```

To obtain the user's alias, we will use the get method.

```js
// Current User's username
export const username = writable("");

user.get("alias").on((v) => username.set(v));
```

We will create a on event function that will update the user's username whenever he or she logs in and out.

---

## Step 3 - Creating Header

Now we'll make a header for our app that will display our username and avatar when we sign up or login in. In the `src` directory, create a new file called `Header.svelte`.

We will import the user database and the username from the user.js file into the script tag. We'll also write a `signout` function that logs the user out and sets the username to an empty string.

```html
<script>
  import { username, user } from "./user";

  function signout() {
    user.leave();
    username.set("");
  }
</script>
```

In the header tag, we will create a conditional block that will display the username along with a unique avatar generated using the [DiceBear API](https://avatars.dicebear.com/) if the username is not an empty string.

```html
<header>
  <h1>🔫 Chat App</h1>
  {#if $username}
  <div class="user-bio">
    <span>Hello <strong>{$username}</strong></span>
    <img src={`https://avatars.dicebear.com/api/human/${$username}.svg`}
    alt="avatar" />
  </div>

  <button class="signout-button" on:click="{signout}">Sign Out</button>
  {/if}
</header>
```

---

## Step 4 - Creating Login form

Now we'll design our login and sign up forms, where users may log in or create new accounts. Make a new file called `Login.svelte` for this purpose.

We will import the user database from the `user.js` file into the script tag. In addition, we will create state variables for `username` and `password`.

Then we'll create two functions: `login` and `signup`. In the login function, we will utilize the `auth` method to validate the user. We will also handle errors and notify the user if any input is incorrect.

We will create a new item in the database using the username and password in the sign up method.

```html
<script>
  import { user } from "./user";

  let username;
  let password;

  function login() {
    user.auth(username, password, ({ err }) => err && alert(err));
  }

  function signup() {
    user.create(username, password, ({ err }) => {
      if (err) {
        alert(err);
      } else {
        login();
      }
    });
  }
</script>
```

Following that, we'll add input fields for the username and password, as well as use the `bind:value` method in Svelte to update the state variables when the input changes.

```html
<label for="username">Username</label>
<input name="username" bind:value="{username}" minlength="3" maxlength="16" />

<label for="password">Password</label>
<input name="password" bind:value="{password}" type="password" />

<button class="login" on:click="{login}">Login</button>
<button class="login" on:click="{signup}">Sign Up</button>
```

---

## Step 5 - Building Chat Component

To begin, we will import various modules and global state variables while developing our chat component, so create a new file called `Chat. svelte` .

```js
import Login from "./Login.svelte";
import ChatMessage from "./ChatMessage.svelte";
import { onMount } from "svelte";
import { username, user } from "./user";
import debounce from "lodash.debounce";
import GUN from "gun";
```

Now we'll declare some new state variables named `newMessage`, which will store the message, the `messages` array, and some minor scrolling feature variables.

```js
const db = GUN();

let newMessage;
let messages = [];

let scrollBottom;
let lastScrollTop;
let canAutoScroll = true;
let unreadMessages = false;
```

Then we'll write some scroll methods that will monitor chat and auto-scroll as new messages arrive.

```js
function autoScroll() {
  setTimeout(() => scrollBottom?.scrollIntoView({ behavior: "auto" }), 50);
  unreadMessages = false;
}

function watchScroll(e) {
  canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
  lastScrollTop = e.target.scrollTop;
}

$: debouncedWatchScroll = debounce(watchScroll, 1000);
```

Next, we'll utilize the `onMount` hook, which will be executed when the component is initialized. It contains a match variable, which functions similarly to a RegEx, and will look for any messages that are less than 3 hours old.

```js
onMount(() => {
  var match = {
    // lexical queries are kind of like a limited RegEx or Glob.
    ".": {
      // property selector
      ">": new Date(+new Date() - 1 * 1000 * 60 * 60 * 3).toISOString(), // find any indexed property larger ~3 hours ago
    },
    "-": 1, // filter in reverse
  };
});
```

Following that, we will retrieve the chat by using the get method, map on it once, and if any data is received, we will create a new variable that will store the same key that will aid in decrypting the data.

Then we will reformat the data as per our convenience in a new message object with keys - who, what and when. In the `who` key we will store the username of the sender, in the `what` key we will store the message after decrypting it using the key and in the `when` key we will store the timestamp of the message.

If a new message is received, we will perform an if conditional block that will store it in the messages array.

```js
db.get('chat')
      .map(match)
      .once(async (data, id) => {
        if (data) {
          // Key for end-to-end encryption
          const key = '#foo';

          var message = {
            // transform the data
            who: await db.user(data).get('alias'), // a user might lie who they are! So let the user system detect whose data it is.
            what: (await SEA.decrypt(data.what, key)) + '', // force decrypt as text.
            when: GUN.state.is(data, 'what'), // get the internal timestamp for the what property.
          };

          if (message.what) {
            messages = [...messages.slice(-100), message].sort((a, b) => a.when - b.when);
            if (canAutoScroll) {
              autoScroll();
            } else {
              unreadMessages = true;
            }
          }
        }
```

Now we will construct a new function `sendMessage` that will allow users to send messages, and then we will insert the message into the database using the put method.

```js
async function sendMessage() {
  const secret = await SEA.encrypt(newMessage, "#foo");
  const message = user.get("all").set({ what: secret });
  const index = new Date().toISOString();
  db.get("chat").get(index).put(message);
  newMessage = "";
  canAutoScroll = true;
  autoScroll();
}
```

We will now create the UI for the chat component, which will loop through all of the messages in the messages array based on the timestamp. In addition, we will create a new input field for users to send messages.

Then we'll add some scrolling elements that will scroll to the bottom when fresh messages arrive.

```html
<div class="container">
  {#if $username}
  <main on:scroll="{debouncedWatchScroll}">
    {#each messages as message (message.when)}
    <ChatMessage {message} sender="{$username}" />
    {/each}

    <div class="dummy" bind:this="{scrollBottom}" />
  </main>

  <form on:submit|preventDefault="{sendMessage}">
    <input
      type="text"
      placeholder="Type a message..."
      bind:value="{newMessage}"
      maxlength="100"
    />

    <button type="submit" disabled="{!newMessage}">💥</button>
  </form>

  {#if !canAutoScroll}
  <div class="scroll-button">
    <button on:click="{autoScroll}" class:red="{unreadMessages}">
      {#if unreadMessages} 💬 {/if} 👇
    </button>
  </div>
  {/if} {:else}
  <main>
    <Login />
  </main>
  {/if}
</div>
```

---

## Step 5 - Create Message Bubble

Now we'll make a chat message bubble, so make a new file called `ChatMessage.svelte` and paste the following code into it:

```html
<script>
  export let message;
  export let sender;

  const messageClass = message.who === sender ? "sent" : "received";

  const avatar = `https://avatars.dicebear.com/api/human/${message.who}.svg`;

  const ts = new Date(message.when);
</script>

<div class="{`message" ${messageClass}`}>
  <img src="{avatar}" alt="avatar" />
  <div class="message-text">
    <p>{message.what}</p>

    <time>{ts.toLocaleTimeString()}</time>
  </div>
</div>
```

---

## Step 6 - Finalizing

We are nearly finished with our app; simply go to `App.svelte` and import the header and chat components into our app component.

```html
<script>
  import Chat from "./Chat.svelte";
  import Header from "./Header.svelte";
</script>

<div class="app">
  <header />
  <Chat />
</div>
```

---

## Conclusion

First and foremost, congratulations 🎉🎉 on making it this far. You have successfully built a decentralized chat app utilizing GUN.js and svelte as the frontend framework.

This article was inspired by a [Fireship](https://www.youtube.com/@Fireship) video:

https://www.youtube.com/watch?v=J5x3OMXjgMc

This article demonstrates what you can build using GUN.js. Feel free to improve on the project by:

- Adding User to User E2E Encryption
- Improving the UI

Thank you for reading!

If you enjoyed this essay and would like to see more on web3, blockchain, and decentralization, make sure to follow and connect with me.

---
