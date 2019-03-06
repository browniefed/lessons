## Intro

Hooks are a way to create reusable logic across your functional React components. They come with a whole new learning curve as you adjust your previous class based knowledge to a hook based knowledge.

One common pattern within React components is listening to an event on the window. My general use case is mouse events.
Commonly an `onMouseDown` event will be registered on the element. You may want to add `onMouseMove` and `onMouseUp` to this element however this will only trigger if the user releases their mouse on top of the element. The solution to this is to register mouse events on the window. We can track the mouse move across the entire page, and know when the user has released their mouse.

## Class solution

One such solution to this issue is with classes. This is how much code has to be written for a single window event listener.
If we supported an event being passed in as a prop rather than hard-coding `mouseup` we would need to write an entire event management system.

```js
class MouseUpHook extends Component {
  componentDidMount = () => {
    // We pass it a function on our class so we don't have to handle if the `onEvent` prop changes on any given render
    window.addEventListener("mouseup", this.handleEvent);
  };
  componentWillUnmount = () => {
    window.removeEventListener("mouseup", this.handleEvent);
  };

  handleEvent = e => {
    this.props.onEvent(e);
  };

  render() {
    return this.props.children;
  }
}
```

## Why it's not ideal

The class solution is less than ideal a couple reasons. The first being having to deal with comparing things manually. In our class solution if we bound to `this.props.onEvent` we'd need to do comparisons in `componentDidUpdate` and remove/re-register listeners. This is a lot of work and bound to have issues.

Second it creates a false hierarchy. When you're just returning null, or directly passing through the children this adds to the hierarchy of your render. However when thinking about elements rendering they are generally associated to what is rendering in the DOM.

To use this `MouseUpHook` we'd do something like this.

```js
<MouseUpHook onEvent={this.handleMouseUp}>
  <div>Other code here</div>
</MouseUpHook>
```

It has nothing to do with the hierarchy of render, but prior to hooks this was our only real solution to wrapping up logic and making it reusable.

## Hooks!

Lets see what this looks like with hooks.

The first step is what `componentDidMount` and `componentWillUnmount` looks like in hooks.

`useEffect` is that answer. It allows us to execute logic in the mounting, updating, and unmounting phases.

```js
useEffect(() => {});
```

No we register a window listener

```js
useEffect(() => {
  window.addEventListener("mouseup", props.onEvent);
});
```

We need to clean up our window listener since hooks are always going to be re-run if you don't specify your dependencies.

So to clean it up we need to return a cleanup function to unregister our function.

```js
useEffect(() => {
  window.addEventListener("mouseup", props.onEvent);

  return () => window.removeEventListener("mouseup", props.onEvent);
});
```

However we might not want this to constantly be registering and un registering listeners every single render.
This is super fast and likely won't cause any performance issues lets add in our proper dependencies for `useEffect`.

```js
useEffect(() => {
  window.addEventListener("mouseup", props.onEvent);

  return () => window.removeEventListener("mouseup", props.onEvent);
}, [props.onEvent]);
```

Perfect, we created all the same logic as our class before using a hook. This is all great for a one-off but lets make it reusable across all our components now.

## Reusable

There isn't much thought to making this reusable besides wrapping it in a function. Which is one of the true powers of hooks.

```js
export const useMouseUp = callback => {
  useEffect(() => {
    window.addEventListener("mouseup", callback);
    return () => window.removeEventListener("mouseup", callback);
  }, [callback]);
};
```

We took our hook, dropped it into a function and now we can use this across any component that need mouse up events.
To take this one step further though we can create a global event listener and start create even more reusable hooks.

If we look at our code we see that `mouseup` could be replaced with `mousemove` or any other event we want to listen to. So lets just receive it as an argument to our function.

```js
export const useWindowEvent = (event, callback) => {
  useEffect(() => {
    window.addEventListener(event, callback);
    return () => window.removeEventListener(event, callback);
  }, [event, callback]);
};
```

We are receiving another prop so we will need to add it in to our dependencies for `useEffect`. Now we can create any number of window events with ease.

```js
export const useGlobalMouseUp = (callback) => {
  return useWindowEvent("mouseup", callback);
};

export const useGlobalMouseMove = (callback) => {
  return useWindowEvent("mousemove", callback);
};
```

## Using the Hook

To use the hooks we call just call them.

```js
export default function CoolComponent() {
  useGlobalMouseMove(e => console.log(e));
  return <div>Other code here</div>;
}
```

We've now drastically reduced our required code needed to listen to a window event, we didn't create a false hierarchy, and now we have a reusable hook to be used across our entire codebase.

## With useCallback

In our previous code you may notice that we are passing in a function to `useGlobalMouseMove`. This is going to cause a new function to be defined every single render and our window listener will once again be constantly registered and cleaned up.

One way to avoid this is `useCallback`. Like `useEffect` it will also take a list of dependencies that are referenced in the callback and return the same function if the dependencies don't change. Dependencies being data and functions that are used with-in the arrow function that is passed to our `useCallback`.

```js
export default function CoolComponent() {
  const callback = useCallback(e => {
    console.log(e);
  }, []);

  useGlobalMouseMove(callback);
  return <div>Other code here</div>;
}
```

This works great until we start to reference props. Consider this code below.

```js
export default function CoolComponent({ type }) {
  const callback = useCallback(() => {
    console.log(type);
  }, []);

  useGlobalMouseMove(callback);
  return <div>Other code here</div>;
}
```

If we are preventing the window from unregistering/re-registering with `useCallback` and have specified that the callback will never change by specifying the `[]` as the `useCallback` dependencies. That means the original callback will have closed over the original `type`.

If a new `type` prop is passed in our callback will be logging the old `type`. This isn't what we want.
So in order to have our window event not register/unregister window events every single re-render, and also have our `useCallback` update correctly we need to specify the dependencies used with in our callback.

```js
export default function CoolComponent({ type }) {
  const callback = useCallback(() => {
    console.log(type);
  }, [type]);

  useGlobalMouseMove(callback);
  return <div>Other code here</div>;
}
```

Our code is now correct. If `type` changes our `useCallback` will return a new function and will then be closing over the correct values to log. If not it'll return the same function that we used before. Additionally our add/remove listeners inside of our `useWindowEvent`.

## Another Solution?

Another solution would be to pass additional dependencies to `useEffect`. Something like this.

```js
export const useWindowEvent = (event, callback, dependencies) => {
  useEffect(() => {
    window.addEventListener(event, callback);
    return () => window.removeEventListener(event, callback);
  }, [event, callback, ...dependencies]);
};
```

If we used it in our `type` prop logging scenario

```js
//useGlobalMouseMove would proxy dependencies
useGlobalMouseMove(() => {
  console.log(type);
}, [type]);
```

One reason I do not like this is it places the dependencies onto a hook that doesn't actually care about the dependencies nor depends on them at all.
The `useWindowEvent` only depends on the `event` being passed in and the `callback`. Those are the only 2 things it cares about changing and needing to re-run.

This is the reason React provides the `useCallback` hook. Create your reusable hooks based upon the dependencies they care about, and use additional provided hooks like `useCallback` to do the dependency comparisons where the dependencies are actually being used.


## Complete

There you have it, a hook you can take and add to your own code. Use the `useEffect` and `useCallback` knowledge to create new hooks that are performant and correct. 

If you want to see some code in action check out this Code Sandbox

[https://codesandbox.io/s/jplmpv2v85](https://codesandbox.io/s/jplmpv2v85?fontsize=14)
