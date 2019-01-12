## Input with Label

```js
import React, { Component, createRef } from "react";
import "./app.css";

class App extends Component {
  state = {
    value: "",
  };

  render() {
    const { value } = this.state;

    return (
      <div className="App">
        <label className="input_container">
          <span className="label">Email</span>
          <input
            className="input"
            value={value}
            name="email"
            onChange={e => this.setState({ value: e.target.value })}
          />
        </label>
      </div>
    );
  }
}

export default App;
```

```css
.input_container {
  position: relative;
  display: flex;
  flex-direction: column;
  margin-top: 16px;
}

.label {
  font-size: 16px;
  line-height: 16px;
  font-family: Helvetica, sans-serif;
}

.input {
  height: 30px;
  width: 100px;
  padding: 8px 4px;
  font-size: 18px;
}
```

## Center our Label

```css
.label {
  font-size: 16px;
  line-height: 16px;
  position: absolute;
  left: 4px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}
```

## Detect Autofill

```css
@keyframes onAutoFillStart {
}

@keyframes onAutoFillCancel {
}
```

```css
.input:-webkit-autofill {
  animation-name: onAutoFillStart;
  transition: background-color 50000s ease-in-out 0s;
}

.input:not(:-webkit-autofill) {
  animation-name: onAutoFillCancel;
}
```

```js
<input
  className="input"
  value={value}
  name="email"
  onAnimationStart={this.handleAutoFill}
  onChange={e => this.setState({ value: e.target.value })}
/>
```

```js
state = {
  value: "",
  hide: false,
};
handleAutoFill = e => {
  this.setState({
    hide: e.animationName === "onAutoFillStart",
  });
};
```

## Hide if Filled

```css
.hide {
  top: -8px;
  left: 0;
  font-size: 12px;
  pointer-events: all;
}
```

```js
const { hide, value } = this.state;
const hideLabel = hide || value;
```

```js
<label className="input_container">
  <span className={`label ${hideLabel ? "hide" : ""}`}>Email</span>
  <input
    className="input"
    value={value}
    name="email"
    onAnimationStart={this.handleAutoFill}
    onChange={e => this.setState({ value: e.target.value })}
  />
</label>
```

## Hide when Focused/Blurred

```js
state = {
  value: "",
  hide: false,
  focus: false,
};
```

```js
render() {
  const { hide, focus, value } = this.state;
  const hideLabel = hide || focus || value;

  return (
    <div className="App">
      <label className="input_container">
        <span className={`label ${hideLabel ? "hide" : ""}`}>Email</span>
        <input
          className="input"
          value={value}
          name="email"
          onAnimationStart={this.handleAutoFill}
          onFocus={() => this.setState({ focus: true })}
          onBlur={() => this.setState({ focus: false })}
          onChange={e => this.setState({ value: e.target.value })}
        />
      </label>
    </div>
  );
}
```

## Ending
