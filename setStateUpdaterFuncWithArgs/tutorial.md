# Setup

First off, check out the [Live Demo](https://codesandbox.io/s/github/browniefed/tmp/tree/setStateRedux)
 so you know what we're building.

Our basic setup is a simple incrementer/decrementer. We have a `value` in state, and 2 buttons to control that value. 

When those buttons are pressed we call either increment or decrement.

```js
import React, { Component } from "react";

class App extends Component {
  state = {
    value: 0,
  };
  increment = () => {};
  decrement = () => {};
  render() {
    return (
      <div>
        <div>{this.state.value}</div>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }
}
```

# The Wrong Way

Here is how many would do this.

```js
this.setState({
  value: this.state.value + 1
});
```

There is theoretically nothing wrong with this at the moment. However once we hit the async world of React where updates don't happen exactly when `setState` is called there is a potential for this to cause issues with your app.

Yes incrementing a number is trivial but apply it to anytime you've used setState.

If you were to call setState twice in a row and reference `this.state.value`. The `value` on `this.state` has not been updated yet.

```js
this.setState({
  value: this.state.value + 1
});
this.setState({
  value: this.state.value + 1
});
```

What this would boil down to is

```js
this.setState({
  value: 1 + 1
});
this.setState({
  value: 1 + 1
});
```
You would have wanted to update to 2 but you're only updating to 1. In order to fix this we need to use `setState` updater functions.

# setState Updater Function

A `setState` updater function is a function passed to `setState`. It is called with both `state` and `props`.
The `state` that it is given is the fully flushed through state from any previous `setState` calls.

```js
this.setState((state, props) => {
  return {
    value: state.value + 1,
  };
});
```

Lets take a look at the example up above using the `setState` updater pattern.

If we did it this way we would get the expected value to be 2. Using an updater function will preserve the order of how state should be applied as well as make sure all previous states are flushed through.

```js
this.setState((state, props) => {
  return {
    value: state.value + 1, // state.value is 0 in this case
  };
});
// Now the previous the setState has flushed through
this.setState((state, props) => {
  return {
    value: state.value + 1, // state.value is 1 in this case because of previous `setState`.
  };
});
```

The result is `2` rather than `1` like it was previously. So now that we have a small fundamental understanding of updater functions lets dive into turning it into a reducer pattern.

# Create Constants

First off at the top of our file we create some constants that will be our actions.

```js
const INCREMENT = "INCREMENT";
const DECREMENT = "DECREMENT";
```

# Create a Reducer

Next we setup a reducer. This is slightly different than a typical redux reducer.
Look closely at the structure. It is a function that returns another function!

```js
const reducer = action => (state, props) => {
  switch (action.type) {
  }
};
```
Here we have a function that takes an argument `action`. When that is called the `action` is in scope of the next function that is returned. This is going to be the function that is passed into `setState`. This is our updater function.

A typical redux reducer would have a signature like below, but in order to work with `setState` it needs to be adjusted slightly.

```js
const reducer = (state, action) => {}
```

# Add Reducer Logic

Now we can add our logic just like any old reducer. We'll do a `switch` on our `action.type`.

```js
const reducer = action => (state, props) => {
  switch (action.type) {
    case INCREMENT:
      return {
        value: state.value + 1,
      };
    case DECREMENT:
      return {
        value: state.value - 1,
      };
    default:
      return null;
  }
};
```

How this varies from a redux reducer is that we don't need to return an object with the state spread into it. A typical return value might look something like this. 

```js
  return {
    ...state,
    value: state.value + 1,
  };
```

Since we are working with React and `setState` whatever we return will be merged into the previous `state` just like doing a normal `setState` call with just an object. So we don't need to return everything.

Finally on handy feature of `setState` and the updater function is if `null` is returned then no re-render will happen. So essentially if our `reducer` is called with something we don't recognize then we won't do anything and we'll tell React we don't want anything to happen either.


# Setup Reducer Updater Functions

To use the reducer function along with React we need to call it with one of the constants we setup, and then pass it into `setState`.

```js
increment = () => {
  this.setState(
    reducer({
      type: INCREMENT,
    }),
  );
};

decrement = () => {
  this.setState(
    reducer({
      type: DECREMENT,
    }),
  );
};
```

If we deconstruct this a little. The reducer is called with our `action`. Which then returns an updater function. Then that is passed into `setState` just like a normal updater function.

```js
const updaterFunction reducer({
  type: INCREMENT,
});
this.setState(updaterFunction);
```

# Pass in Data

We can pass more than just `type` we can pass other data too. So if we wanted our `increment` to increment by 2 rather than 1 we need to add an `amount` to the object. 

```js
increment = () => {
  this.setState(
    reducer({
      type: INCREMENT,
      amount: 2,
    }),
  );
};
```

Then adjust our `reducer` to reference our `action.amount`.

```js
case INCREMENT:
  return {
    value: state.value + action.amount,
  };
```

## Ending

Why we would want to do this? Well it externalizes your state management into a testable function. It also sets us up for proper state updates when dealing with async rendering in React. You don't have to go full `reducer` pattern, but moving your updates to their own functions can make testing your data updating logic much easier.

[Live Demo](https://codesandbox.io/s/github/browniefed/tmp/tree/setStateRedux)
