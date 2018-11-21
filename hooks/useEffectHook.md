## Intro

Hooks are reacts response to adding functionality of classes to stateless components. It does more than just add the functionality and flexibility of class components, it adds reusablity of logic across your codebase.

## Setup

To get setup we'll create two hooks to help test out how the `useEffect` hook operates. One will trigger a simple state update to re-render. The other will control unmounting of our child component we'll create.

```js
import React, { useEffect, useState } from "react";

const App = () => {
  const [trigger, setTrigger] = useState(true);
  const [mounted, setMount] = useState(true);
  return (
    <div>
      <button onClick={() => setTrigger(!trigger)}>Trigger Update</button>
      <button onClick={() => setMount(!mounted)}>Toggle Mount</button>
    </div>
  );
};
```

## Create a Component

Create a stateless component called `Position`. We'll just be logging out the mouse position so for this component we can just return `null` and not render anything.

```js
const Position = () => {
  return null;
};
```

## Creating a useEffect Hook

Our `logMousePosition` function will log out the `x/y` of the mouse position as it moves.

We call `useEffect` and pass in a function. This function will run every single time our component re-renders. We attach a window listener on `mousemove` and log the mouse position. 

At the moment we have an issue. If you re-render this component our `useEffect` will be called again. That will attach another mouse listener to the window and now we're logging the mouse position twice.

```js
const logMousePosition = e => {
  console.log({
    x: e.clientX,
    y: e.clientY,
  });
};


const Position = () => {
  useEffect(() => {
    window.addEventListener("mousemove", logMousePosition);
  });

  return null;
};
```

## Clean Up Phase

We want to avoid attaching an endless amount of event listeners so we'll use the clean up phase of the effect. Every time the component re-renders all effects will be re-run. To clean up the side effects you must return a function.

In our case we want to cleanup by calling `removeEventListener`.

```js
useEffect(() => {
  window.addEventListener("mousemove", logMousePosition);
  console.log("Created");
  return () => {
    console.log("Cleaned up");
    window.removeEventListener("mousemove", logMousePosition);
  };
};
```

If you run this code everything will work, however when you trigger an update. You can see with our console logs that the clean up phase is run. We don't need to keep attaching and detaching the listeners.

In order to register just once we need to pass in an empty array as the second argument to `useEffect`.

```js
useEffect(() => {
  window.addEventListener("mousemove", logMousePosition);
  console.log("Created");
  return () => {
    console.log("Cleaned up");
    window.removeEventListener("mousemove", logMousePosition);
  };
}, []);
```
This typically is used to control whether or not the `useEffect` needs to be re-applied. This array is diffed from the original creation of the effect and the new one being passed in. It will diff the array (just like it does the virtual DOM) and decide if it needs to re-apply the effect.

Passing in an empty array tells React to diff, however there is nothing different between each render so the effect will only be run once. Be aware though, if you are calling a function from props, or relying on props inside the effect you will need to pass them into the array to re-apply the effect.

For example if you were relying on an `id` inside the effect you'd pass in `[props.id]` and React will take care of re-applying it for you.

## Toggle Mount

Finally to prove that React is cleaning up our effects when the component unmounts you can toggle the rendering of our `Position` component. You can see that when you unmount it, `Cleaned up` will be logged and our mouse movements will no longer be logged.

```js
const App = () => {
  const [trigger, setTrigger] = useState(true);
  const [mounted, setMount] = useState(true);
  return (
    <div>
      <button onClick={() => setTrigger(!trigger)}>Trigger Update</button>
      <button onClick={() => setMount(!mounted)}>Toggle Mount</button>

      {mounted ? <Position /> : null}
    </div>
  );
};
```