## Introduction



## Get Some Images

[Unsplash](https://unsplash.com/) is the place to be when it comes to getting beautiful free images.

If you want access to the ones I picked you can grab them from the github folder from the repo here: [Our Images](https://github.com/codedailyio/teach/tree/instagramSwiper/public)

I went for images that were all roughly the same dimensions so when it came time to render them we could use a singular width. This

## Render Them Side By Side

Lets dive into the basic outline of our application. We define width/height up front, however this could be driven off of a dynamic container size in the future. To simplify the concept we'll focus on static dimensions.

We need to build our DOM structure in such a way that the our outer container is like a window into our inner container.

Our inner container is the one that will be scrolling and contain all our images. We then use the outer container to create a `700x400` window into the inner container which we'll refer to as the `swiper`.

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

To accomplish this window into the swiper our `main` wrapper needs to be set to `overflow: 'hidden'`. Then our style dimensions that we set will create the window to look in. Our `swiper` will be a flex container that has the `overflow-x` set to visible. It'll automatically lay our images out in a row.

Finally we setup our future animations by saying that our transition property will be the transform style, and then give the browser a few hints about what's going to change so it can optimize our animation/movement.

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

The `object-fit` is for our images, this will contain them so they maintain their aspect ratio while shrinking to fit inside of our `700x400` window.

## Get Scrolling

On our `main` container we apply the `onWheel` event. This allows us to capture any wheel/scroll events when the mouse happens to be over our `div`. So even though there is nothing to scroll and no scroll bar we can still capture scrolling events.

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

To save off the current movement of our scroll we add a new bit into state and set it to `0`. This will track our offset and starting at `0` is just us saying start at the beginning of the scrollable container.

```js
state = {
  imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
  movement: 0,
};
```

We want to support more than scrolling we want to support touch movements. So we will need to unify everything into a singular function. We only need the `deltaX` aka the horizontal change for that particular event.

The `deltaX` could be positive or negative depending on which direction the user is scrolling. This `deltaX` will generally be a small number.

```js
handleWheel = e => {
  this.handleMovement(e.deltaX);
};
```

We're going to need to reference the previous state so we'll need to use a callback reference style of `setState`. We first calculate a few pieces of data. 

The first is the total length of the images we have, then we create our `nextMovement` which is the next position to offset our `swiper`. 

We need to setup some constraints so that as you scroll left and or reach the end you don't scroll to far and that you stop when you reach the ends. To do this we check if our future `nextMovement` is less than `0`, if it is we just set it to `0`.

Then if the nextMovement is greater than the total number of our images multiplied by the image width. If it is more than we set it to the maximum.

```js
handleMovement = delta => {
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

Finally we apply our `movement` to our to our swiper. We need to multiply by `-1` so that the `movement` left/right actually works with `translateX` and moves things left/right.

```js
const { movement} = this.state;

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

In order to work on mobile devices we need to handle touches. We need to handle the start, move, and end events from touches.

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

We can't simply get the delta of a touch on the web so we need to keep track of our last touch position.

```js
class App extends Component {
  lastTouch = 0;
  state = {
    imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
    movement: 0,
  };
}
```

Upon touch start we save off where the first finger touch was located at.
```js
handleTouchStart = e => {
  this.lastTouch = e.nativeEvent.touches[0].clientX;
};
```

Then we calculate the delta from the X movement. We can subtract the current touch location to where our previous touch was at. Once we have the delta we can update our `lastTouch` with the current touch.

Finally we can then call our `handleMovement` with our `delta` and when the touch is done we can clear our previous touch to `0`.

```js
handleTouchMove = e => {
    const delta = this.lastTouch - e.nativeEvent.touches[0].clientX;
    this.lastTouch = e.nativeEvent.touches[0].clientX;

    this.handleMovement(delta);
  };
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
