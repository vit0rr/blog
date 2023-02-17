---
title: Introduction to React.useEffect hook
date: '6-30-2022'
tags: ['Tutorials']
draft: false
summary: Introduction to React.useEffect hook
images: /static/images/banners/difference-between-state.jpg
---

### The logic inside React components

_This article is based on and has code examples imported from [here](https://beta-reactjs-org-git-effects-fbopensource.vercel.app/learn/synchronizing-with-effects)._

You need to be familiar with two types of logic:

- Rendering code is what lives on the top of your component. This is where you take the props and state, transform them, and return the JSX that you want to see on screen. [Keep components pure](https://beta-reactjs-org-git-effects-fbopensource.vercel.app/learn/keeping-components-pure). It should only calculate the result, nothing anything else.

- [Event handlers](https://developer.mozilla.org/en-US/docs/Web/Events/Event_handlers) are functions that do something rather than just calculate them. An event handlers contain [side effects](<https://en.wikipedia.org/wiki/Side_effect_(computer_science)#>) that can change the program state and are caused by specific user action, like a button click or typing.

But this isn't enough. A ChatRoom, for example, must connect to the chat server whenever it's visible on the screen. Connecting to a chat server it's a side effect then can't happen during the rendering the component. But, there is no single event like a click or typing that caused ChatRoom to be displayed. And now?

Effects let you specify side effects rendering themselves and not by a particular event like a click. Send some message in the chat is an event because is directly caused by some user action, like the click on the button or something like that. Setting up a server is an effect because the connection happens with which interaction causes the component to appear. It's worth you knowing that Effects run at the end of the [rendering process](https://beta-reactjs-org-git-effects-fbopensource.vercel.app/learn/render-and-commit) after the screen updates. Here is a nice moment to synchronize the React component with some external system like a network.

## Maybe you don't need an effect

Don't run to add effects to your components. Effects are used to "step out" of your code and sync with some external system. If your effect only adjusts some states based on other states, maybe you don't need an effect.

## How to write an effect

1. Declare and effect
2. Specify the effect dependencies
3. Add cleanup or stop if needed

## 1: Declare an effect

First of all, you need to import the useEffect hook in your React component:

```js
import { useEffect } from 'react'
```

At in the top level of your component, call it and put some code inside your effect:

```js
function MyComponent() {
  useEffect(() => {
    // Code
  })
  return <div />
}
```

Every time the component `MyComponent` renders, React will update the screen _and then_ run the code inside of useEffect. **The code inside effect only will run after React load the component**.

Now let's see how to use effect with some external system. Consider a React componente named `<VideoPlayer>` that have a prop isPlaying.

```js
<VideoPlayer isPlaying={isPlaying} />
```

The component VideoPlayer renders a built-in browser `<video>` tag:

```js
function VideoPlayer({src, isPlaying}) {
    return <video src={src} >;
}
```

The `<video>` tag does not have an `isPlaying` prop. `play()` and `pause()` method looks the only way to control it. We need to synchronize the value of `isPlaying` prop. You need to synchronize the value of `isPlaying` prop, which tells whether the video should currently be playing, with imperative calls like play() or pause().

Now you need to be familiar with [ref](https://beta-reactjs-org-git-effects-fbopensource.vercel.app/learn/manipulating-the-dom-with-refs) on React.

```javascript
import { useState, useRef, useEffect } from 'react'

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null)

  if (isPlaying) {
    ref.current.play() // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause() // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false)
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>{isPlaying ? 'Pause' : 'Play'}</button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  )
}
```

In this case, just calling the play or pause function during rendering is the wrong way. Why? Cause trying to do something in DOM during rendering some component in react is impossible. Remember that [rendering should be a pure calculation](https://beta-reactjs-org-git-effects-fbopensource.vercel.app/learn/keeping-components-pure) of JSX and shouldn't contain some side effects that modify the DOM.

Is simple. How can you modify something that does not exist yet? Impossible.

The solution is wrap our side effect with `useEffect` to move it out of the rendering calculation.

```javascript
// https://play-pause.vercel.app/
import { useState, useRef, useEffect } from 'react'

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null)

  useEffect(() => {
    if (isPlaying) {
      ref.current.play()
    } else {
      ref.current.pause()
    }
  })

  return <video ref={ref} src={src} loop playsInline width={420} />
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false)
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>{isPlaying ? 'Pause' : 'Play'}</button>{' '}
      <br />
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  )
}
```

This example keeps simple to understand the "external system" to synchronize with react States.

## Step 2: Specify the effect dependencies

By default, like that code that we learned, effects run after every render. But not always is what we want.

- Sometimes the synchronizing is not always instant. For example, you do not want to reconnect to the chat server on every keystroke.

- Sometimes, we want that some animation should happen only in the first render, for the first time.

An example to demonstrate the issue, with a few `console.log` and an input that updates the parent component's state.

```js
import { useState, useRef, useEffect } from 'react'

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null)

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()')
      ref.current.play()
    } else {
      console.log('Calling video.pause()')
      ref.current.pause()
    }
  })

  return <video ref={ref} src={src} loop playsInline />
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false)
  const [text, setText] = useState('')
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>{isPlaying ? 'Pause' : 'Play'}</button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  )
}
```

Every keystroke, will call the `console.log`. Now think that instead of a `log`, the function that calls is a connection to some server, a fetch, or CSS animation. Chaotic.

To fix this, we need to specify an array of dependencies. Like this:

```js
useEffect(() => {
  // code
}, [])
```

```js
import { useState, useRef, useEffect } from 'react'

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null)

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()')
      ref.current.play()
    } else {
      console.log('Calling video.pause()')
      ref.current.pause()
    }
  }, [isPlaying])

  return <video ref={ref} src={src} loop playsInline />
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false)
  const [text, setText] = useState('')
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>{isPlaying ? 'Pause' : 'Play'}</button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  )
}
```

The array can contain multiple dependencies.

## Step 3: Add cleanup or stop if needed

Now consider a different example: our ChatRoom component needs to connect to the chat server when it appears. We are giving a `createConnection()` API that returns an object with `connect()` and `disconnect()`. How to connect when it is displayed to the user?

Let's start building the effect logic:

```js
useEffect(() => {
  const connection = createConnection()
  connection.connect()
}, [])
```

My dependency array is empty because our effect does not have props or states. This tells React to only load this code when the component mounts.

Our code is:

```js
// App.js
import { useState, useEffect } from 'react'
import { createConnection } from './chat.js'

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection()
    connection.connect()
  }, [])
  return <h1>Welcome to the chat!</h1>
}
```

```js
// chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('Connecting...')
    },
    disconnect() {
      console.log('Disconnected.')
    },
  }
}
```

But, if you check your console on your browser, will see "Connecting" printed twice. To help us, React remounts every component once immediately after its initial mount. Log twice helps us to notice the real issue: our code doesn't close the connection when the component unmounts. And this is bad. Why?

Think if you have a project with many components to render in different tabs. When you go to another page, the connection was not closed, and you accumulate so many connections/connected requests. **You need to close the connection when the component unmounts**.

To fix this, let's return a cleanup function from our effect:

```js
useEffect(() => {
  const connection = createConnection()
  connection.connect()
  return () => {
    connection.disconnect()
  }
}, [])
```

React will call our cleanup function each time before the effect runs again, and one final time when the component unmounts.

```js
// App.js
import { useState, useEffect } from 'react'
import { createConnection } from './chat.js'

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection()
    connection.connect()
    return () => connection.disconnect()
  }, [])
  return <h1>Welcome to the chat!</h1>
}
```

```js
// chat.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('Connecting...')
    },
    disconnect() {
      console.log('Disconnected.')
    },
  }
}
```

And in browser console:

1. "Connecting..."
2. "Disconnected."
3. "Connecting..."

This is the correct behavior in development.
This behavior says that when React unmounts our component, our connection is "Disconnected". And when mount again, "Connecting...".

> I have on me all the dreams of the world.
