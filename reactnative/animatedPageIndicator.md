## Introduction

Adding small effects to an application can make you a huge impact on the users experience and enjoyment using your application. One such small touch is animating paging indicators inside of a carousel.

There are plenty of techniques to animate a moving indicator from the outside but what about animating on the inside!

In React Native we want to limit the work sent over the bridge and also limit `setState` during an animation. We're going to accomplish all of this with out a single `setState`.

## Create our Circles

We'll be rendering 8 circles, this is just derived from an `items` array but in your actual application this would be driven by your own pages.

We loop over and create a `circle` View for each page we have.

```js
import React, { Component } from "react";
import { StyleSheet, Text, Animated, View, TouchableOpacity } from "react-native";

const items = [0, 1, 2, 3, 4, 5, 6, 7];

class App extends Component {
  render() {
    return (
      <View style={styles.container}>
        <View style={styles.background}>
          {items.map(i => {
            return <View style={styles.circle} key={i} />;
          })}
        </View>
      </View>
    );
  }
}
```

Our `background` wrapping all of our circles needs to be set to `flexDirection: 'row'` so these circles are laid out in a row. If you need this vertical instead you wouldn't need this.

Our circle is a defined with, and half that width for the `borderRadius` to make it a circle. The key piece here is to add in `overflow: hidden` so that as our movement indicators move later they aren't visible.

```js
const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  background: {
    flexDirection: "row",
  },
  circle: {
    width: 40,
    height: 40,
    backgroundColor: "#ddd",
    borderRadius: 20,
    marginRight: 4,
    overflow: "hidden",
  },
});
```

![](https://images.codedaily.io/lessons/reactnative/pageIndicators/no_mover.png)

## Add Movement Indicators

Now we need to add in our movement indicators. Each circle will have its own movement indicator. We render our mover `View` inside each circle.

```js
return (
  <View style={styles.circle} key={i}>
    <View style={[styles.mover]} />
  </View>
);
```

In React Native all items are relative to their parent so we can toss an `absolute: "position"` on here and make the styling match the outer circle.

```js
  mover: {
    position: "absolute",
    top: 0,
    left: 0,
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: "tomato",
  },
```

![](https://images.codedaily.io/lessons/reactnative/pageIndicators/all_filled.png)

## Animate Indicators

First we need to create an `Animated.Value` to store our current index. 

```js
state = {
  index: new Animated.Value(0),
};
```

```js
const { index } = this.state;
```

To avoid having to store an integer and using `setState` to figure out our math offset for each indicator we can use `Animated` math.

Our math will determine how far we need to `translateX` which will control our left to right movement.


We need to take our current `index` and subtract the index of the circle we're rendering. Then use `Animated.multiply` and multiply by the circle width of `40`.

These math functions will apply even during animations, so as our `index` changes and animates the circle will update in partial amounts.

We can then pass that into a `translateX` transform style. 

```js
{
  items.map(i => {
    const translateX = Animated.multiply(Animated.subtract(index, i), 40);
    const transform = {
      transform: [{ translateX }],
    };
    return (
      <View style={styles.circle} key={i}>
        <Animated.View style={[styles.mover, transform]} />
      </View>
    );
  });
}
```
![](https://images.codedaily.io/lessons/reactnative/pageIndicators/single_circle.png)


## Click to Move

To avoid having to use `setState` and also track the current page index we use Animated dynamic tracking animations.

This allows for us to setup an animation that automatically executes when the animated value changes.

So we create an `Animated.Value` called `track` that starts at `0`. 
Then we pass that `track` value into our `toValue` for our `Animated.timing`. Whenever our `track` animated value changes the animation will then immediately kick off.

This will then animate our `index` and thus animate our paging indicators. 

```js
state = {
  index: new Animated.Value(0),
  track: new Animated.Value(0),
};
componentDidMount = () => {
  Animated.timing(this.state.index, {
    duration: 500,
    toValue: this.state.track,
    useNativeDriver: true,
  }).start();
};
```

Now we switch our paging indicators over to be `TouchableOpacity` and when ever pressed we do a `setValue` on our `track` animated value to trigger our animation.

```js
return (
  <TouchableOpacity style={styles.circle} key={i} onPress={() => this.state.track.setValue(i)}>
    <Animated.View style={[styles.mover, transform]} />
  </TouchableOpacity>
);
```

## Ending

Now you can animate paging indicators without using `setState` and taking advantage of Animated's dynamic value tracking animation system.

![](https://images.codedaily.io/lessons/reactnative/pageIndicators/click_move.gif)
