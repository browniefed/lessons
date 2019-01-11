## Introduction

## Create our Circles

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

## Add Movement Indicators

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

## Animate Indicators

```js
state = {
  index: 0,
};
```

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

## Automate Movement

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

## Ending
