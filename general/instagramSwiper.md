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

To save off the current movement of our scroll we add a new bit into state and set it to `0`. This will track our offset and starting at `0` is us saying "start at the beginning of the scrollable container".

```js
state = {
  imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
  currentIndex: 0,
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

We need to setup some constraints so that as you scroll left and or reach the end you don't scroll to far and that you stop when you reach the ends. To do this we check if our future `nextMovement` is less than `0`, if it is we set it to `0`.

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
const { movement } = this.state;

<div
  className="swiper"
  style={{
    transform: `translateX(${movement * -1}px)`,
  }}
>
  {imgs.map(src => {
    return <img key={src} src={src} width="100%" height="100%" />;
  })}
</div>;
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

As we move our slider forward we'll need to detect what images we have landed on so we can normalize our slide and snap to a particular image.
For our wheel we need to modify `handleWheel`. We save off a timeout to `wheelTimeout` and clear it on each event. As the user scrolls and eventually stops the events will stop flowing in. We give ourselves `100ms` before we then attempt to snap the image to a specific index.

```js
handleWheel = e => {
  clearTimeout(this.wheelTimeout);
  this.handleMovement(e.deltaX);
  this.wheelTimeout = setTimeout(() => this.handleMovementEnd(), 100);
};
```

For the touch events all we need to do is directly call our end movement function.

```js
handleTouchEnd = () => {
  this.handleMovementEnd();
  this.lastTouch = 0;
};
```

We need to add in the `currentIndex` because with out knowing where we start we won't be able to tell which direction the user has actually moved.

```js
state = {
  imgs: ["/img1.jpg", "/img2.jpg", "/img3.jpg", "/img4.jpg"],
  currentIndex: 0,
  movement: 0,
};
```

To make it all work we need to do a few calculations. The first is to get the final `movement` position and divide it by the total image width. This will give us the image and the percentage of any other image we're looking at.

For example if we scrolled past half the first image this number would be `.5`, or if you scrolled entirely past the first image and a little into the 3rd image this number would be something like `1.2`.

To get the partial piece of an image we use the `modulus` which when dividing by the proceeding number will return the remainder. If we divide by `1` like we do here this will always give us the remainder, aka the decimal place. In our case from the previous example this would return `.5`, or `.2`.

Then we get our final `endingIndex` which is the full number of images we have bypassed. So in the `.5` case we haven't bypassed any images so we're at `0`. As we scroll past `1` full image the `endingIndex` would be 1.

Finally we calculate out the delta. The `deltaInteger` is how we are going to detect if the direction of our movement. If we start at an index of `0` and end at `1` then we have moved 1 full image forward.

But if we started at the second image (aka `currentIndex: 1`) then scroll backwards our `endingIndex` is `0`. So our delta is now `-1`.

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

We will start with the naive assumption that `nextIndex` is the total number of images the user has scrolled by.

We'll want buttons in the future to move backwards/forwards so again we'll consolidate our movement/transition logic to a singular function called `transitionTo`. We save off the `currentIndex`, then also calculate the offset that we need to move towards which is the index multiplied by image width.

```js
transitionTo = index => {
  this.setState({
    currentIndex: index,
    movement: index * IMG_WIDTH,
  });
};
```

Now we need to do some logic. Assuming that our `endingIndex` is how many images we scrolled past, if we scrolled to `1.9` this would be bypassing the whole first image, and 90% of the second image.

It's clear the user is now looking at the 3rd image Without more logic we would snap backwards and look at the second image.

We first do a check if we're moving forward which is if our `deltaInteger` is positive or `0`. Then we need to check how much of any other image we're looking at.

Instagram is very aggressive at moving a user towards the next image so if we're looking at another image at least `10%` we want to snap to that photo. So in our case if we will add `1` to our `nextIndex`.

```js
if (deltaInteger >= 0) {
  if (endPartial >= 0.1) {
    nextIndex += 1;
  }
}

this.transitionTo(nextIndex);
```

## Moving Backwards

Moving backwards is much like moving forward but our percentage of what image we are looking at will be `.9` since we'll be looking at the right side of the image.

So if our `deltaInteger` is negative we set our `nextIndex` to the current index minus the number of images we have moved. This will give us the next image even if we're only looking at a small sliver of it.

We need to do the same thing as before but slightly differently. We check if the partial is looking at more than `90%` of the image. If it's at `95%` only a small sliver is visible so we add `1` and thus snap back to the photo that is most visible to the user.

```js
 else if (deltaInteger < 0) {
  nextIndex = currentIndex - Math.abs(deltaInteger);
  if (endPartial > 0.9) {
    nextIndex += 1;
  }
}
```

## Transitions

Now in order to transition from a current position to another we need to define a transition duration. The duration defaults to `0` so as we scroll things happen instantly. If we then apply a duration and a new movement position at the same time then things will animate and snap to.

We then setup a timeout to clear our duration once we hit the end. We could detect when the animation ends and kill state but there is a possibility that a user interrupts our snap. So we need to handle that in our other functions. We convert our decimal to an amount of milliseconds by multiplying by `100`. So `.5s` would be equivalent to `500ms`.

```js
transitionTo = (index, duration) => {
  this.setState({
    currentIndex: index,
    movement: index * IMG_WIDTH,
    transitionDuration: `${duration}s`,
  });

  this.transitionTimeout = setTimeout(() => {
    this.setState({ transitionDuration: "0s" });
  }, duration * 100);
};

componentWillUnmount = () => {
  clearTimeout(this.transitionTimeout);
};
```

At the end of our `handleMovementEnd` we take the minimum value and pass that to our duration. We have a maximum of `.5s` but attempt to calculate how much of the image is totally visible and make our animation snappier if only a small bit is visible.

```js
this.transitionTo(nextIndex, Math.min(0.5, 1 - Math.abs(endPartial)));
```

To finalize the snap if the user then interrupts the snap animation we clear our transition timeout, and for the next movement we set our duration to `0s` so we're back to instant movement.

```js
handleMovement = delta => {
  clearTimeout(this.transitionTimeout);

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
      transitionDuration: "0s",
    };
  });
};
```

Finally we need to apply our `transitionDuration` to our `swiper`.

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

The final bit of code is adding in our next/previous buttons as well as hiding them appropriately.

We pull off and determine a few variables like the total number of images we have and the maximum movement we can translate the swiper.

```js
const { currentIndex, movement, transitionDuration, imgs } = this.state;
const maxLength = imgs.length - 1;
const maxMovement = maxLength * IMG_WIDTH;
```

For the `back` movement if we are currently at `0` we are at the beginning of the container. This means we don't want the back button to appear so we hide it. Since this is all being updated live and instant it's a good value to determine visiblity of our buttons off of.

When clicked the button will then move us to our `currentIndex - 1` over half a second.

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
```

The same concept goes for our next button however we'll only render it if the container hasn't been scrolled to it's maximum amount. Then if clicked we move to the `currentIndex + 1` over half a second.

```js
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

We make a singular `move` class for our button and use the vertical positioning technique by setting `top: 50%` and then translate back upwards `-50%`. This is also why we need our `main` class wrap to be `relative`.

Then we bust out our separate classes to position them `left/right` away from the edge.

```css
.move {
  display: flex;
  position: absolute;
  width: 40px;
  height: 40px;
  top: 50%;
  transform: translateY(-50%);
  border-radius: 20px;
  background-color: rgba(255, 255, 255, 0.5);
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border: 0;
}

.back {
  left: 5px;
}
.next {
  right: 5px;
}
```

## Ending
