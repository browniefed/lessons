## Intro

Many websites provide a mechanism to verify a user based upon a text message. This can be a random code usually 4-6 numbers. It doesn't really matter as we'll build an input that can do it all, but one common interaction is auto-advancing inputs when you type.

This on the surface can look trivial but can be complex when you are dealing with user interactions like pasting, deleting, and focus management.

This tutorial was inspired by logging into Stripe and seeing how they handled their text code verification.

![](https://images.codedaily.io/lessons/general/verify_input/stripe_example.png)

## Setup 6 Rectangles

6 is arbitrary this will work with any amount as long as we base our calculations off of the length of the array.

```js
const CODE_LENGTH = new Array(6).fill(0);
```

We start with a bit of state. We will hold our value in a single string and use the `split` method to turn it into an array. We will loop over our `CODE_LENGTH` just so we can grab the index and render in our display value.

```js
class App extends Component {
  state = {
    value: "",
  };
  render() {
    const { value } = this.state;
    const values = value.split("");

    return (
      <div className="App">
        <div className="wrap">
          {CODE_LENGTH.map((v, index) => {
            return <div className="display">{values[index]}</div>;
          })}
        </div>
      </div>
    );
  }
}

export default App;
```

Our display value will be divs that have their border right the same color and size as the wrap. The wrapping div will provide the border look, and the display pieces will provide the separation on the inside.

We will use `:last-child` to remove the border from the last display so we don't have a double border on the right side from the display and wrap border.

We also use display flex to allow us to easily center our text content inside of our display.

```css
.display {
  border-right: 1px solid rgba(0, 0, 0, 0.2);
  width: 32px;
  height: 58px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  position: relative;
}
.display:last-child {
  border-right: none;
}

.wrap {
  border: 1px solid rgba(0, 0, 0, 0.2);
  display: inline-block;
  position: relative;
  display: flex;
}
```

![](https://images.codedaily.io/lessons/general/verify_input/input_borders.png)

## Setup Our Input

We need to get access to our `input` via a ref so we can control it's focus.

```js
class App extends Component {
  input = React.createRef();
}
```

We render our input, and one important piece is to put `value=""`. The reason for this is to indicate to React that it's a controlled input, and that even if people type into we never want to display a value.

If we left that off the text input would be uncontrolled and would have user input displayed. Instead of displaying the content in the input we have our value split from state and rendered in separate divs.

```js
<div className="wrap" onClick={this.handleClick}>
  <input
    value=""
    ref={this.input}
    className="input"
    style={{
      width: "32px",
      top: "0px",
      bottom: "0px",
      left: "0px",
    }}
  />
</div>
```

We will render our input absolutely because we will want to control the movement of it later so that we don't have to fake a cursor but instead move the input to the active cell.

Also we remove the outline and some other default as we will recreate these later with separate styling.

```css
.input {
  position: absolute;
  border: none;
  font-size: 32px;
  text-align: center;
  background-color: transparent;
  outline: none;
}
```

## Handle Focus/Blur

On important piece is controlling focus and blur. We will need to track whether or we have focus so we can display appropriate outlines. So we add a boolean `focused` value to state, and attach an `onFocus` and `onBlur` to our input.

```js
state = {
  value: "",
  focused: false,
};

handleClick = () => {
  this.input.current.focus();
};
handleFocus = () => {
  this.setState({ focused: true });
};
handleBlur = () => {
  this.setState({
    focused: false,
  });
};
```

```js
const { value, focused } = this.state;
```

Not only that we need to handle what happens when the user clicks on the outer wrap. If it's clicked we need to focus on the input. So we add an `onClick` and call focus on our `ref` that we had created earlier.

Calling focus directly like this will then trigger our `onFocus` handler so we don't need to set any state in the `handleClick` method.

```js
<div className="wrap" onClick={this.handleClick}>
  <input
    value=""
    ref={this.input}
    onFocus={this.handleFocus}
    onBlur={this.handleBlur}
    className="input"
    style={{
      width: "32px",
      top: "0px",
      bottom: "0px",
      left: "0px",
    }}
  />
</div>
```

## Move Our Input

One important piece is moving the input. As the user types we will move it starting from position 0 through to the full length of the code. We do this so as the user types the input can display in the cell with a real cursor flashing.

We do some math to determine the selected index to multiply by `32` which is just the width of each display cell.

If our current length of typed in code is less than the total possible values then we return the `values.length` which is just the length of characters users have typed in.

If we hit the point where the user has typed in all possible characters then we just return the total possible length of characters. This limits the input to always be visible in the last square and no further.

```js
const selectedIndex = values.length < CODE_LENGTH.length ? values.length : CODE_LENGTH.length - 1;

<input
  value=""
  ref={this.input}
  onFocus={this.handleFocus}
  onBlur={this.handleBlur}
  className="input"
  style={{
    width: "32px",
    top: "0px",
    bottom: "0px",
    left: `${selectedIndex * 32}px`,
  }}
/>;
```

## Handle Typing

Now when the user types we will have our change event fire. Generally you would update state with `e.target.value` however since we put our `value=""` the value will always be the singular character that the user typed.

We are reference previous state so we need to use the callback `setState` style. Since this will be called later and React cleans up and reuses synthetic events we need to save off the typed in value otherwise `e.target.value` will be `undefined`.

```js
handleChange = e => {
  const value = e.target.value;

  this.setState(state => {
    if (state.value.length >= CODE_LENGTH.length) return null;
    return {
      value: (state.value + value).slice(0, CODE_LENGTH.length),
    };
  });
};
```

Also the `handleChange` method could be potentially called with more than one character if the user pasted in the number. So to combat any possibility of over pasting past the maximum character limit we will combine the current state, new incoming value, then `slice` it down to the maximum length.

If a user pasted in `8` characters we would grab `0` to `6` in our case and only update the state with the first 6 characters.

Also if the current length is greater than or equal to maximum code length we return null. This will prevent typing in more than the allowed amount or characters.

![](https://images.codedaily.io/lessons/general/verify_input/input_noshadow.gif)

## Fake Outline

Accessibility is important to think about and so we will need to re-create the outline that would normally display when the user has focus. We could add in the outline but choose to do it this way for better design/aesthetic purposes.

To accomplish this we need to know if a current cell is selected, or if everything is all filled in.

If it's currently selected we want to render our outline, or if all numbers have been filled in then we want to render the outline in the last input. We do that by comparing the index we are looping over directly to the length of the characters that have been typed in.

Then to check if it's filled we compare length of values typed in and also check if the loop has gotten to the final input.

```js
{
  CODE_LENGTH.map((v, index) => {
    const selected = values.length === index;
    const filled = values.length === CODE_LENGTH.length && index === CODE_LENGTH.length - 1;

    return (
      <div className="display">
        {values[index]}
        {(selected || filled) && focused && <div className="shadows" />}
      </div>
    );
  });
}
```

We also need to check if the user is currently focused. If they aren't focused on the input then we don't want to render our outline.

To render the outline we render another div positioned absolutely. The `display` is positioned `relative` so it will render and cover each individual character display.
Then using the `box-shadow` style we can create that has `0` offset in bother directions, no blur, but a spread-radius. The spread radius will make a 4px border on the outside of the display.

```css
.shadows {
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  box-shadow: 0 0 0 4px rgba(58, 151, 212, 0.28);
}
```

## Handle Delete

We will need to handle deleting manually because the input doesn't have any value. So there is no way to detect a value change to remove a number. We can grab the key from the `onKeyUp` callback and check if it's backspace. If the user is deleting then we can use `slice` to remove the last character from our `value` on state.

```js
handleKeyUp = e => {
  if (e.key === "Backspace") {
    this.setState(state => {
      return {
        value: state.value.slice(0, state.value.length - 1),
      };
    });
  }
};

<input
  value=""
  ref={this.input}
  onFocus={this.handleFocus}
  onBlur={this.handleBlur}
  onKeyUp={this.handleKeyUp}
  className="input"
/>
```

## Handle Edge Cases

One final edge case is to handle when the user has fully typed in the amount of characters. They can type in 5 characters and be focused on the 6th, but once they type in the final character the `selectedIndex` will move the input to the right.

However if we have all the numbers, the input will be trying to collect a character for stuff we don't want/need. Typically in React you'd just un-render the input but if the user clicks on our wrapping div we need to have access to the `input` so we can focus. Otherwise after the user typed in all the numbers they'd have no way to delete them if they got them wrong.

```js
const hideInput = !(values.length < CODE_LENGTH.length);
```

To handle this we will check if the user has typed in all characters than just hide the input using opacity.

```js
<input
  value=""
  ref={this.input}
  onChange={this.handleChange}
  onKeyUp={this.handleKeyUp}
  onFocus={this.handleFocus}
  onBlur={this.handleBlur}
  className="input"
  style={{
    width: "32px",
    top: "0px",
    bottom: "0px",
    left: `${selectedIndex * 32}px`,
    opacity: hideInput ? 0 : 1,
  }}
/>
```

## Ending

That's it, we now have an input that auto-progresses whenever you type it in and stops when the maximum character limit is reached.

![](https://images.codedaily.io/lessons/general/verify_input/input_complete.gif)