## Explanation

## Setup

```js
import React, { Component } from "react";

class App extends Component {
  state = {
    width: 0,
    height: 0
  };
  componentDidMount() {
    const { width, height } = this.r.getBoundingClientRect();
    this.setState({
      width,
      height
    });
  }
  render() {
    return (
      <div>
        <h2 ref={r => (this.r = r)}>
          {this.state.width} x {this.state.height}
        </h2>
      </div>
    );
  }
}
```

## Expectation

My previous expectation was that you would see a flash on the screen of `0 x 0`.

## Actual

What actually happens.

## End

[Live Demo](https://codesandbox.io/s/z3pkq2059p)