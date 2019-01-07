## Introduction



## Get Some Images

[Unsplash](https://unsplash.com/) is the place to be when it comes to getting beautiful free images.

If you want access to the ones I picked you can grab them from the github folder from the repo here: [Our Images](https://github.com/codedailyio/teach/tree/instagramSwiper/public)

I went for images that were all roughly the same dimensions so when it came time to render them we could use a singular width. This

## Render Them Side By Side

```js
import React, { Component } from "react";
import "./app.css";

const IMG_WIDTH = 700;
const IMG_HEIGHT = 400;

class App extends Component {
  state = {
    imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
  };
  render() {
    const { imgs } = this.state;

    return (
      <div className="App">
        <div
          className="main"
          style={{
            width: `${IMG_WIDTH}px`,
            height: `${IMG_HEIGHT}px`,
          }}
        >
          <div className="swiper">
            {imgs.map(src => {
              return <img key={src} src={src} width="100%" height="100%" />;
            })}
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```

```css
.main {
  background-color: #000;
  overflow: hidden;
  position: relative;
}

.swiper {
  display: flex;
  overflow-x: visible;
  transition-property: transform;
  will-change: transform;
}

img {
  object-fit: contain;
}
```

## Get Scrolling

```js
<div
  className="main"
  style={{
    width: `${IMG_WIDTH}px`,
    height: `${IMG_HEIGHT}px`,
  }}
  onWheel={this.handleWheel}
>
```

```js
state = {
  imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
  movement: 0,
};
```

```js
handleWheel = e => {
  this.handleMovement(e.deltaX);
};
```

```js
handleMovement = delta => {
  clearTimeout(this.wheelTimeout);

  this.setState(state => {
    const maxLength = state.imgs.length - 1;
    let nextMovement = state.movement + delta;

    if (nextMovement < 0) {
      nextMovement = 0;
    }

    if (nextMovement > maxLength * IMG_WIDTH) {
      nextMovement = maxLength * IMG_WIDTH;
    }

    return {
      movement: nextMovement,
    };
  });
};
```

```js
<div
  className="swiper"
  style={{
    transform: `translateX(${movement * -1}px)`,
  }}
>
  {imgs.map(src => {
    return <img key={src} src={src} width="100%" height="100%" />;
  })}
</div>
```

## Get Touching

```js
<div
  className="main"
  style={{
    width: `${IMG_WIDTH}px`,
    height: `${IMG_HEIGHT}px`,
  }}
  onTouchStart={this.handleTouchStart}
  onTouchMove={this.handleTouchMove}
  onTouchEnd={this.handleTouchEnd}
  onWheel={this.handleWheel}
>
```

```js
class App extends Component {
  lastTouch = 0;
  state = {
    imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
    movement: 0,
  };
}
```

```js
handleTouchStart = e => {
  this.lastTouch = e.nativeEvent.touches[0].clientX;
};
handleTouchMove = e => {
  const delta = e.nativeEvent.touches[0].clientX - this.lastTouch;
  this.lastTouch = e.nativeEvent.touches[0].clientX;

  this.handleMovement(delta * -1);
};
handleTouchEnd = () => {
  this.lastTouch = 0;
};
```

## Moving Forward

```js
handleWheel = e => {
  this.handleMovement(e.deltaX);
  this.wheelTimeout = setTimeout(() => this.handleMovementEnd(), 100);
};
```

```js
handleTouchEnd = () => {
  this.handleMovementEnd();
  this.lastTouch = 0;
};
```

```js
handleMovementEnd = () => {
  const { movement, currentIndex } = this.state;

  const endPosition = movement / IMG_WIDTH;
  const endPartial = endPosition % 1;
  const endingIndex = endPosition - endPartial;
  const deltaInteger = endingIndex - currentIndex;

  let nextIndex = endingIndex;
};
```

```js
transitionTo = index => {
  this.setState({
    currentIndex: index,
    movement: index * IMG_WIDTH,
  });
};
```

```js
if (deltaInteger >= 0) {
  if (endPartial >= 0.1 && (deltaInteger >= 1 || currentIndex >= endingIndex)) {
    nextIndex += 1;
  }
}

this.transitionTo(nextIndex);
```

## Moving Backwards

```js
 else if (deltaInteger < 0) {
  nextIndex = currentIndex - Math.abs(deltaInteger);
  if (endPartial > 0.9) {
    nextIndex += 1;
  }
}
```

## Transitions

```js
transitionTo = (index, duration) => {
  this.setState({
    currentIndex: index,
    movement: index * IMG_WIDTH,
    transitionDuration: `${duration}s`,
  });

  this.transitionTimeout = setTimeout(() => {
    this.setState({ transitionDuration: "0s" });
  }, 10);
};
```

```js
this.transitionTo(nextIndex, Math.min(0.5, 1 - Math.abs(endPartial)));
```

```js
const { movement, transitionDuration, imgs } = this.state;

<div
  className="swiper"
  style={{
    transform: `translateX(${movement * -1}px)`,
    transitionDuration: transitionDuration,
  }}
>
  {imgs.map(src => {
    return <img key={src} src={src} width="100%" height="100%" />;
  })}
</div>;
```

## Previous/Next Buttons

```js
const { currentIndex, movement, transitionDuration, imgs } = this.state;
const maxLength = imgs.length - 1;
const maxMovement = maxLength * IMG_WIDTH;
```

```js
{
  movement !== 0 && (
    <button
      className="back move"
      onClick={() => {
        this.transitionTo(currentIndex - 1, 0.5);
      }}
    >
      ←
    </button>
  );
}
{
  movement !== maxMovement && (
    <button
      className="next move"
      onClick={() => {
        this.transitionTo(currentIndex + 1, 0.5);
      }}
    >
      →
    </button>
  );
}
```

## Ending
