## Intro

When building out a custom bottom action sheet many techniques implemented by libraries require you to measure the bottom content. There is a different technique that we will dive into. The basics are positioning a view off screen. Then with our other content (in our case a bottom card that slides up) we can translate negative height of the screen to bring the inner view into visible area.

This technique is showing how it could be used with an a bottom sheet, but you could do a similar technique for a centered modal, etc.

## Open Button

We'll need an open button. This will likely be some beautiful button somewhere in your app but a simple text button is all we need.

```js
return (
  <View style={styles.container}>
    <TouchableOpacity onPress={this.handleOpen}>
      <Text>Open</Text>
    </TouchableOpacity>
  </View>
);
```

For our styling we'll have our button just centered right in the middle of the screen.

```js
const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

![](https://images.codedaily.io/lessons/reactnative/bottomsheet/open_button.png)

## Backdrop

We are setting up a backdrop that will cover the content of the app. It's not necessary to add in a background color but we will add one in. We use the `StyleSheet.absoluteFill` which is a helper from React Native. It's equivalent of just setting the view absolutely and setting `top/left/right/bottom` all to `0` to cover everything.

```js
<View style={styles.container}>
  <TouchableOpacity onPress={this.handleOpen}>
    <Text>Open</Text>
  </TouchableOpacity>

  <Animated.View style={[StyleSheet.absoluteFill, styles.cover]} />
</View>
```

```js
cover: {
  backgroundColor: "rgba(0,0,0,.5)",
},
```

![](https://images.codedaily.io/lessons/reactnative/bottomsheet/backdrop.png)

## The Action Sheet

Now comes in the technique for setting up our bottom action sheet. Our `sheet` style is positioned absolutely. We set it to `100%` height so it'll be the entire height of our screen, and then we set the `top` to the window height.

This gets the screen height and moves our wrapping view all the way off the screen.

```js
<View style={[styles.sheet]}>
  <Animated.View style={[styles.popup]}>
    <TouchableOpacity>
      <Text>Close</Text>
    </TouchableOpacity>
  </Animated.View>
</View>
```

The key piece here for the content to appear at the bottom is to set `justifyContent` to `flex-end`.

This will set all of our content rendering to the bottom of the view. This means no matter what size our action sheet content is it will always be at the end.

Finally we add in our close button. Your real content would go here. To give the view some height I set our bottom popup to be a minimum height of `80` but it isn't necessary for this technique to work.

```js
  sheet: {
    position: "absolute",
    top: Dimensions.get("window").height,
    left: 0,
    right: 0,
    height: "100%",
    justifyContent: "flex-end",
  },
  popup: {
    backgroundColor: "#FFF",
    marginHorizontal: 10,
    borderTopLeftRadius: 5,
    borderTopRightRadius: 5,
    minHeight: 80,
    alignItems: "center",
    justifyContent: "center",
  },
```

![](https://images.codedaily.io/lessons/reactnative/bottomsheet/overlay.png)

## Setup Animation

Now we get to our animation. Everything will be driven from a single animated value so that our animation is 100% reversible. We don't need to manage separate animations, it's all driven off of a single `0` for close or `1` for open and all animated states are interpolated.

```js
state = {
  animation: new Animated.Value(0),
};
```

For our open/close we'll use `Animated.timing` with a duration. We are animating opacity and translation so that means we can apply `useNativeDriver: true` and all of our animations will run on the native side rather than driven in the JS world. This sets us up for very performant animations.

```js
handleOpen = () => {
  Animated.timing(this.state.animation, {
    toValue: 1,
    duration: 300,
    useNativeDriver: true,
  }).start();
};
handleClose = () => {
  Animated.timing(this.state.animation, {
    toValue: 0,
    duration: 200,
    useNativeDriver: true,
  }).start();
};
```

Go find our buttons and apply the `onPress` method with the appropriate handler.

```js
<TouchableOpacity onPress={this.handleOpen}>
  <Text>Open</Text>
</TouchableOpacity>

<TouchableOpacity onPress={this.handleClose}>
  <Text>Close</Text>
</TouchableOpacity>
```

## Animate Backdrop

If we only translate the popup then our backdrop will never cover our application content. Meaning the content will be interactive and touchable. We don't want this.

One trick to get it to appear is use `interpolation` with a cliff. We set a `0` to `.01` cliff for a virtually instant animation. We need to be sure and `clamp` our interpolation or as our animation progresses for `0` all the way to `1` our `translateY` on our backdrop will continue moving and won't stop at `0`.

Now with our backdrop instantly moved to cover everything we can animate the `opacity`. We can start it at `.01` and finish at `.5` which is half way through the animation. Start it at `.01` guarantees that the backdrop is in place before we start fading it in. That way the user won't see a flash of darkness as the backdrop moves into place.

```js
render() {
  const screenHeight = Dimensions.get("window").height;

  const backdrop = {
    transform: [
      {
        translateY: this.state.animation.interpolate({
          inputRange: [0, 0.01],
          outputRange: [screenHeight, 0],
          extrapolate: "clamp",
        }),
      },
    ],
    opacity: this.state.animation.interpolate({
      inputRange: [0.01, 0.5],
      outputRange: [0, 1],
      extrapolate: "clamp",
    }),
  };

//JSX DOWN HERE
}
```

Now lets apply the backdrop animation style to our backdrop animated view.

```js
<Animated.View style={[StyleSheet.absoluteFill, styles.cover, backdrop]}>
```

## Animate Action Sheet

Now our action content is rendering inside of our `popup` and inside of our backdrop. We have positioned it at the very end so that means it needs to overcome the entire screen height for it to display.

We setup our interpolation on our animation to start at `0.01` to ensure that our backdrop is in place by the time it attempts to animate into view. Also the resting position of our view will be at the `flex-end`. Then we animate the entire height of the screen (negatively) so it's moving from the bottom all the way to the top.

```js
const slideUp = {
  transform: [
    {
      translateY: this.state.animation.interpolate({
        inputRange: [0.01, 1],
        outputRange: [0, -1 * screenHeight],
        extrapolate: "clamp",
      }),
    },
  ],
};
```

Now apply our slide up animation to our popup bottom sheet animated view.

```js
<Animated.View style={[styles.popup, slideUp]}>
  <TouchableOpacity onPress={this.handleClose}>
    <Text>Close</Text>
  </TouchableOpacity>
</Animated.View>
```

## Add Something Fun

Just as an example we can render a horizontal `ScrollView` that renders random colors to swipe through. You can easily replace it with any other scrollable actions you desire.

```js
import React, { Component } from "react";
import { View, Text, StyleSheet, ScrollView } from "react-native";

const randomHsl = () => `hsla(${Math.random() * 360}, 100%, 50%, 1)`;
const cards = Array(20).fill(0);

class Scroller extends Component {
  render() {
    return (
      <ScrollView horizontal style={styles.scroll}>
        {cards.map((v, index) => {
          return <View key={index} style={[styles.card, { backgroundColor: randomHsl() }]} />;
        })}
      </ScrollView>
    );
  }
}

const styles = StyleSheet.create({
  scroll: {
    height: 300,
  },
  card: {
    height: "100%",
    width: 200,
  },
});

export default Scroller;
```

Import it then just render inside of our popup. That's it.

```js
import Scroller from "./scroller";

<Animated.View style={[styles.popup, slideUp]}>
  <TouchableOpacity onPress={this.handleClose}>
    <Text>Close</Text>
  </TouchableOpacity>
  <Scroller />
</Animated.View>
```

![](https://images.codedaily.io/lessons/reactnative/bottomsheet/scroller.png)

## Ending

This technique is a simple concept that can be extended to other things like modals, dropdown/popup notifications, or anything else you accessible from the bottom.

![](https://images.codedaily.io/lessons/reactnative/bottomsheet/bottomsheet.gif)
