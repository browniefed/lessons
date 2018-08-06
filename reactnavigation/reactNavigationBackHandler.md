## Intro

The Android back button adds an additional navigation option that is crucial to manage when developing an application. Commonly a button is rendered in the top left to navigate backwards, or utilizing gestures. Android adds an additional option with an actual hardware/virtualized button.

If not handled your users may potentially lose information they've entered into your app. Not only that you may want to control the back navigation based upon the state of your application.

![](https://images.codedaily.io/lessons/reactnavigation/AndroidBackPrompt.gif)

## App Structure

We'll start with a basic setup already. We'll start by using `createStackNavigator` to create a stack of screens that can be navigated to. In our case we just have 2, the Home screen which is the default and the profile screen.

`App.js`

```js
import React from "react";
import { StyleSheet, Text, View } from "react-native";
import { createStackNavigator } from "react-navigation";

import HomeScreen from "./home";
import ProfileScreen from "./profile";

const Navigation = createStackNavigator({
  Home: HomeScreen,
  Profile: ProfileScreen,
});

export default class App extends React.Component {
  render() {
    return <Navigation />;
  }
}
```

The home screen is just a way to navigate to another screen with the application. By default React Navigation will handle the Android back button for you, however we'll need to override the defaults. If you're at the top of the stack and press the android back button the application will close. If you've navigated within the stack anywhere then the screen will pop.

`Home.js`

```js
import React, { Component } from "react";
import { View, Text, StyleSheet, TouchableOpacity } from "react-native";

class HomeScreen extends Component {
  static navigationOptions = {
    title: "Home",
  };
  render() {
    return (
      <View>
        <TouchableOpacity onPress={() => this.props.navigation.push("Profile")}>
          <Text>Go To Profile</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

export default HomeScreen;
```

By default if the android back button was pressed here the stack would pop and would navigate back to the home screen.

Here in the profile we'll simulate what it would be like to have a form that is editable. We won't worry about rendering a form but most forms/profiles would have a flag. That flag will control the editing of the user/form. Additionally you may want to control saving, or validation based upon the users actions.

`Profile.js`

```js
import React, { Component } from "react";
import { View, Text, StyleSheet, TouchableOpacity, Alert } from "react-native";

class ProfileScreen extends Component {
  static navigationOptions = {
    title: "Profile",
  };
  state = {
    editing: false,
  };
  render() {
    const { editing } = this.state;
    return (
      <View>
        <TouchableOpacity onPress={() => this.setState({ editing: !editing })}>
          <Text>Toggle Editing {editing ? "Off" : "On"} </Text>
        </TouchableOpacity>
      </View>
    );
  }
}

export default ProfileScreen;
```

## Creating a Back Component

To simplify logic across your application we'll create a `HandleBack` component. All it will do is `render` the children provided but we'll use the higher order component `withNavigation` to be able to get access to the current state of our entire application navigation.

```js
import React, { Component } from "react";
import { withNavigation } from "react-navigation";
import { BackHandler } from "react-native";

class HandleBack extends Component {
  render() {
    return this.props.children;
  }
}

export default withNavigation(HandleBack);
```

## Navigation Events

React Navigation has life cycle hooks. These listeners all us to hook into the life cycle of our apps navigation. When registered it'll tell us when the current screen focuses, as well as when the current screen blurs as you navigate towards a different screen.

We can use these events to know when to listen and unlisten. When a screen is active we want to listen to it, when it loses focuses we don't want to listen to it anymore.

We want to use the current screen as the logic for how to handle the Android back button being pressed.

Also we'll need to unlisten to our listeners if the component is ever unmounted. When you navigate through your application your screens aren't unmounted, they are merely hidden. This is why listening to the active screen is crucial.

```js
import React, { Component } from 'react';
import { withNavigation } from 'react-navigation';

class HandleBack extends Component {

  constructor(props) {
    super(props);
    this.didFocus = props.navigation.addListener('didFocus', payload =>

    );
  }

  componentDidMount() {
    this.willBlur = this.props.navigation.addListener('willBlur', payload =>

    );
  }

  onBack = () => {
    return this.props.onBack();
  };

  componentWillUnmount() {
    this.didFocus.remove();
    this.willBlur.remove();
  }

  render() {
    return this.props.children;
  }
}

export default withNavigation(HandleBack);
```

## Handling Back

Now we need to leverage `BackHandler` from `react-native`. We add another event listener for `hardwareBackPress`. We do this in the constructor versus on mount because we need to capture the screen actually gaining focus. In the constructor it hasn't rendered yet. If we call this in mount and register the `didFocus` we would miss the event.

With our `BackHandler` we also need to remove the listener upon blurring, as well as when the component unmounts.

At any point that the `BackHandler` listeners return `true` this indicates to React Navigation that the hardware back button has been handled already so don't do anything. In our case we directly return from the `onBack` prop call. So the individual screens get to determine whether or not they have "handled" the hardware back button.

```js
import React, { Component } from "react";
import { withNavigation } from "react-navigation";
import { BackHandler } from "react-native";

class HandleBack extends Component {
  constructor(props) {
    super(props);
    this.didFocus = props.navigation.addListener("didFocus", payload =>
      BackHandler.addEventListener("hardwareBackPress", this.onBack),
    );
  }

  componentDidMount() {
    this.willBlur = this.props.navigation.addListener("willBlur", payload =>
      BackHandler.removeEventListener("hardwareBackPress", this.onBack),
    );
  }

  onBack = () => {
    return this.props.onBack();
  };

  componentWillUnmount() {
    this.didFocus.remove();
    this.willBlur.remove();
    BackHandler.removeEventListener("hardwareBackPress", this.onBack);
  }

  render() {
    return this.props.children;
  }
}

export default withNavigation(HandleBack);
```

## Prompt

Now that we have the ability to hook into the lifecycle of our navigation and handle back button presses we need to use that component. Inside of our Profile screen we can wrap our entire app in our `HandleBack` component we created and pass in an `onBack` call.

```js
  render() {
    const { editing } = this.state;
    return (
      <HandleBack onBack={this.onBack}>
        <View>
          <TouchableOpacity onPress={() => this.setState({ editing: !editing })}>
            <Text>Toggle Editing {editing ? "Off" : "On"} </Text>
          </TouchableOpacity>
        </View>
      </HandleBack>
    );
  }
```

In our `onBack` call we'll check out `this.state.editing`. If it's `false` meaning we're not editing we'll return `false`. This tells React Navigation that even though we registered a `BackHandler` we haven't handled it so go ahead and do what you would normally do. In this case it will navigate backwards to the home page.

Otherwise if the user is editing we return `true`. This tells React Navigation we've handled the back button press so don't do anything. In our case we show an `Alert` prompt. If the user presses `cancel` nothing should happen. The prompt closes and the user can continue editing their information.

If they press `Go Home` we can then manually trigger a navigation event. In our case it's to do the normal routine and just go back.

```js
onBack = () => {
  if (this.state.editing) {
    Alert.alert(
      "You're still editing!",
      "Are you sure you want to go home with your edits not saved?",
      [
        { text: "Keep Editing", onPress: () => {}, style: "cancel" },
        { text: "Go Home", onPress: () => this.props.navigation.goBack() },
      ],
      { cancelable: false },
    );
    return true;
  }

  return false;
};
```

## Ending

Handling the back button is one thing, however you still need to manage the actual `back` button in the header or gestures that are enabled by default on iOS. Many developers plan for what they can see and render, however often forget about interactions with hardware buttons on Android.

[Live Demo here](https://snack.expo.io/@codedaily/androidbackhandlingreactnav)

![](https://images.codedaily.io/lessons/reactnavigation/AndroidBackPrompt.gif)