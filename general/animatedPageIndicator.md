## Introduction

Adding small effects to an application can make you a huge impact on the users experience and enjoyment using your application. One such small touch is animating paging indicators inside of a carousel.

There are plenty of techniques to animate a moving indicator from the outside but what about animating on the inside!

Lets dive in and explore how to build this in React, and then convert it to hooks.

![](https://images.codedaily.io/lessons/general/animatedIndicator/movingindicators.gif)

## Create our Circles

Lets get going by creating a series of circles. This would generally be the amount of pages you are needing to page through but I've chosen an arbitrary number (8).

```js
import React, { Component } from "react";
import "./app.css";

const items = [0, 1, 2, 3, 4, 5, 6, 7];

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="background">
          {items.map(i => {
            return <div className="circle" />;
          })}
        </div>
      </div>
    );
  }
}

export default App;
```

To achieve side by side circles we give our `background` class a `display: flex` which automatically defaults to `flex-direction: row`.

Our circle is a defined size, with a little margin to the right to space each circle out. One key piece to not is that we are setting this to `position: relative` and `overflow: hidden`.

When our movement indicator is rendered inside later we'll need to contain it's position rendering to each individual circle and hide it as it moves.

```css
.background {
  display: flex;
}

.circle {
  width: 40px;
  height: 40px;
  background-color: #ddd;
  position: relative;
  border-radius: 20px;
  margin-right: 4px;
  overflow: hidden;
}
```

![](https://images.codedaily.io/lessons/general/animatedIndicator/just_circles.png)

## Add Movement Indicators

Now we add in our movement indicators. Rather than having a singular item we create movement indicators inside of each circle.

```js
{
  items.map(i => {
    return (
      <div className="circle">
        <div className="mover" />
      </div>
    );
  });
}
```

These are identically sized to the wrapping circle but adding `position: absolute` and `left: 0; top: 0` allows us to cover the circle behind it exactly.

```css
.mover {
  position: absolute;
  left: 0;
  top: 0;
  width: 40px;
  height: 40px;
  border-radius: 20px;
  background-color: tomato;
}
```

![](https://images.codedaily.io/lessons/general/animatedIndicator/all_filled.png)

## Animate Indicators

Now comes the logic of our indicator. We need to have some state so we can track the current index we're looking at.

```js
state = {
  index: 0,
};
```

We define our transition which is our inner indicator moving. We'll be translating it using `transform` so we indicate that when `transform` we'll take `.5s` to complete the transition.

```css
.mover {
  position: absolute;
  top: 0;
  left: 0;
  width: 40px;
  height: 40px;
  border-radius: 20px;
  background-color: tomato;
  transition: transform 0.5s ease;
}
```

We adjust our `translateX` based upon the current selected index we're looking at compared to the index of the circle we're rendering.

When our page is selected we want our `translateX` to be at `0` so that it's fulling cover the background circle. So we can do a little math and when the current `index` of the page minus the current rendering circle is `0` then we will have a `translateX(0)`.

Otherwise everything else is multiplied by the size of the circle and offset.

One thing to point out is we never cap the movement at `40` or `-40`. So if the rendering circle is at index `2` but our selected page is at `6` that means we're 4 pages away.

The circle will be offset at `-160px` but what this will get us is the ability to animate multiple pages and each circle will appear to transition in and out.

```js
render() {
  const { index } = this.state;

  return (
    <div className="App">
      <div className="background">
        {items.map(i => {
          return (
            <div className="circle">
              <div
                className="mover"
                style={{
                  transform: `translateX(${(index - i) * 40}px)`,
                }}
              />
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

![](https://images.codedaily.io/lessons/general/animatedIndicator/single_filled.png)

## Automate Movement

Now lets automate our movement since we haven't actually hooked this up to anything. In our `componentDidMount` we'll update our `state` and increment it by 1.

Each time we update our `transition` will kick off because our `translateX` on each page indicator has changed.

We add a bit of logic so that once our current `index` is at the end we change the direction it moves to an increment of `-1`. Once we're back to the start we switch back to a positive movement of `1`.

```js
increment = 1;
componentDidMount = () => {
  setInterval(() => {
    this.setState(state => {
      if (state.index >= items.length - 1) {
        this.increment = -1;
      } else if (state.index <= 0) {
        this.increment = 1;
      }

      return {
        index: state.index + this.increment,
      };
    });
  }, 500);
};
```

![](https://images.codedaily.io/lessons/general/animatedIndicator/movingindicators.gif)

## Convert to Hooks

The Component version we just built contain 3 concepts. Life cycles, mutable references and state. React has a concept called hooks that allows us to add these concepts to functional components.

So first we switch over our class component to a functional component.

```js
const App = () => {
  return (
    <div className="App">
      <div className="background">
        {items.map(i => {
          return (
            <div className="circle">
              <div
                className="mover"
                style={{
                  transform: `translateX(${(index - i) * 40}px)`,
                }}
              />
            </div>
          );
        })}
      </div>
    </div>
  );
};
```

First lets tackle state. The only state we had was the current index, so we import `useState` from React.

When we call it the first time the component mounts it takes an initial value, our initial value is `0`. This will return an array of 2 pieces. The value at array index `0` is the value of the state, and the second is an updater. The updater is equivalent to `setState` but for that specific hook.

```js
import React, { useState } from "react";
const [index, setIndex] = useState(0);
```

Next we had `componentDidMount` to setup our interval. The `useEffect` hook adds this concept to our functional components.
When the component mounts this code will run, so our interval is created and then we can reference our `setIndex` to update the index every `500ms`.

The return value from our `useEffect` hook is our cleanup and when we want to clean up our effect we need to `clearInterval` since that is the side effect that was created in our `useEffect` hook.

Finally because we want this only run once and not clear our interval if an update happens we need to pass `[]` as the second argument to `useEffect`. This is an array for React to compare if anything changes. If it does then it runs the cleanup and re-runs the effect. Passing an empty array tells React to only run this effect once, and clean up when the component unmounts.

```js
import React, { useState, useEffect } from "react";

useEffect(() => {
  let interval = setInterval(() => {
    if (index >= items.length - 1) {
      increment = -1;
    } else if (index <= 0) {
      increment = 1;
    }

    setIndex(index + increment);
  }, 500);
  return () => clearInterval(interval);
}, []);
```

The increment previously was a mutable reference on the class. Generally in React `refs` are use exclusively for getting references to DOM/Elements. The `useRef` hook creates a mutable reference that can be used for more than just getting references to the DOM.

We can store mutable data that doesn't belong on state, but we need to hang around across re-renders. The value to mutate is stored on the `current`. So if we want to access the value or update it we need to use `increment.current`;

```js
import React, { useState, useEffect, useRef } from "react";

const increment = useRef(1);

useEffect(() => {
  let interval = setInterval(() => {
    if (index >= items.length - 1) {
      increment.current = -1;
    } else if (index <= 0) {
      increment.current = 1;
    }

    setIndex(index + increment.current);
  }, 500);
  return () => clearInterval(interval);
}, []);
```

All together it looks like this.

```js
const App = () => {
  const [index, setIndex] = useState(0);
  const increment = useRef(1);
  useEffect(() => {
    let interval = setInterval(() => {
      if (index >= items.length - 1) {
        increment.current = -1;
      } else if (index <= 0) {
        increment.current = 1;
      }

      setIndex(index + increment.current);
    }, 500);
    return () => clearInterval(interval);
  }, []);
  return (
    <div className="App">
      <div className="background">
        {items.map(i => {
          return (
            <div className="circle">
              <div
                className="mover"
                style={{
                  transform: `translateX(${(index - i) * 40}px)`,
                }}
              />
            </div>
          );
        })}
      </div>
    </div>
  );
};
```

## Ending

You now have the ability to render some really cool animated pagination indicators! Check them out live on [Code Sandbox](https://codesandbox.io/s/github/codedailyio/teach/tree/animatedPageIndicator/?module=%2Fsrc%2FApp.js)


![](https://images.codedaily.io/lessons/general/animatedIndicator/multiple.gif)

