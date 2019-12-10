# rn-meteor

[![react-native-meteor](http://img.shields.io/npm/dm/@xvonabur/react-native-meteor.svg)](https://www.npmjs.org/package/@xvonabur/react-native-meteor) [![npm version](https://badge.fury.io/js/%40xvonabur%2Freact-native-meteor.svg)](https://badge.fury.io/js/%40xvonabur%2Freact-native-meteor) [![Dependency Status](https://david-dm.org/xvonabur/react-native-meteor/status.svg)](https://david-dm.org/xvonabur/react-native-meteor)

This project was adapted from [react-native-meteor](https://github.com/inProgress-team/react-native-meteor) by inProgress Team to be more up to date and focused. This documentation has been revised to be more coherent, dependencies have been updated, and the API has been brought more in-line with Meteor.

If you are moving from that package to this one, you may find certain parts are missing that were deemed to be outside the scope of this package (ListViews), were removed/deprecated in meteor core or are outdated methods of connecting Tracker with your componenets (connectMeteor, composeWithTracker and createContainer).

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [@xvonabur/react-native-meteor](#xvonaburreact-native-meteor)
  - [Supporting The Project](#supporting-the-project)
  - [Installation And Setup](#installation-and-setup)
    - [Android](#android)
  - [Example Usage](#example-usage)
- [Reactive Variables](#reactive-variables)
- [API](#api)
  - [Subscriptions](#subscriptions)
  - [Collections](#collections)
  - [Meteor.is/Environment/](#meteorisenvironment)
  - [DDP connection](#ddp-connection)
    - [Meteor.connect(url, options)](#meteorconnecturl-options)
      - [Meteor.ddp](#meteorddp)
    - [Meteor.disconnect()](#meteordisconnect)
  - [Meteor Methods](#meteor-methods)
  - [Additional Packages](#additional-packages)
    - [ReactiveDict](#reactivedict)
    - [Accounts](#accounts)
    - [React Meteor Data](#react-meteor-data)
- [Contribution](#contribution)

<!-- /TOC -->

## Installation And Setup

```sh
$ npm i --save vforvasile/rn-meteor
```

### Android

Add the following permission to your AndroidManifest.xml file for faster reconnects to the DDP server when your device reconnects to the network.

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

If running an android emulator you have to forward the port of your meteor app.

```shell
$ adb reverse tcp:3000 tcp:3000
```

## Example Usage

```javascript
import React, { Component } from "react";
import { View, Text } from "react-native";
import Meteor, { withTracker } from "react-native-meteor";
import TodosCollection from "../api/todo";

Meteor.connect("ws://192.168.X.X:3000/websocket"); //do this only once

class App extends Component {
  renderRow(todo) {
    return <Text>{todo.title}</Text>;
  }
  render() {
    const { todos, todosReady } = this.props;

    return (
      <View>{todosReady ? todos.map(renderRow) : <View>Loading...</View>}</View>
    );
  }
}

export default withTracker(params => {
  return {
    todosReady: Meteor.subscribe("todos").ready(),
    todos: TodosCollection.find({ done: false }, { sort: { createdAt: -1 } })
  };
})(App);
```

---

# Reactive Variables

These variables can be used inside `withTracker`. They will be populated into your component if they change.

- [Meteor.subscribe()](http://docs.meteor.com/#/full/meteor_subscribe)
- Mongo.Collection(collectionName, options)
  - [.find(selector, options)](http://docs.meteor.com/#/full/find)
  - [.findOne(selector, options)](http://docs.meteor.com/#/full/findone)
- [Meteor.user()](http://docs.meteor.com/#/full/meteor_user)
- [Meteor.userId()](http://docs.meteor.com/#/full/meteor_userid)
- [Meteor.loggingIn()](http://docs.meteor.com/#/full/meteor_loggingin)
- [Meteor.status()](http://docs.meteor.com/#/full/meteor_status)
- [ReactiveDict()](https://atmospherejs.com/meteor/reactive-dict)

---

# API

## Subscriptions

- [Meteor.subscribe()](http://docs.meteor.com/#/full/meteor_subscribe)

```javascript
import Meteor, { Tracker } from "@xvonabur/react-native-meteor";

const handle = Meteor.subscribe("xvonabur.friends");

Tracker.autorun(() => {
  if (handle.ready()) {
    console.log("Subscription ready...");
    handle.stop();
  }
});
```

## Collections

- Mongo.Collection(collectionName, options)
  - [.insert(doc, callback)](http://docs.meteor.com/#/full/insert)
  - [.update(id, modifier, [options], [callback])](http://docs.meteor.com/#/full/update)
  - [.remove(id, callback(err, countRemoved))](http://docs.meteor.com/#/full/remove)

These methods work offline. That means that elements are correctly updated offline, and when you reconnect to ddp, Meteor calls are taken care of.

You need pass the `cursoredFind` option when you get your collection if you want to use cursor-like method:

```javascript
import { Mongo } from "@xvonabur/react-native-meteor";

new Mongo.Collection("collectionName", { cursoredFind: true });
```

Or you can simply use `find()` to get an array of documents. The option default to false for backward compatibility. Cursor methods are available to share code more easily between a react-native app and a standard Meteor app.

## Meteor.is/Environment/

Keeping in line with Meteor's API, `isClient` is provided as well as a `isReactNative`, similar to how Meteor provides `isCordova` when code is running in a cordova build. `isServer` and `isCordova` are not provided as they will still be falsey when checking. These properties allow for code reuse across your codebases.

- Meteor.isClient - True
- Meteor.isReactNative - True

## DDP connection

### Meteor.connect(url, options)

Connect to a DDP server. You only have to do this once in your app.

_Arguments_

- `url` **string** _required_
- `options` **object** Available options are :
  - autoConnect **boolean** [true] whether to establish the connection to the server upon instantiation. When false, one can manually establish the connection with the Meteor.ddp.connect method.
  - autoReconnect **boolean** [true] whether to try to reconnect to the server when the socket connection closes, unless the closing was initiated by a call to the disconnect method.
  - reconnectInterval **number** [10000] the interval in ms between reconnection attempts.

#### Meteor.ddp

Once connected to the ddp server, you can access every method available in [ddp.js](https://github.com/mondora/ddp.js/).

- Meteor.ddp.on('connected')
- Meteor.ddp.on('added')
- Meteor.ddp.on('changed')

### Meteor.disconnect()

Disconnect from the DDP server.

## Meteor Methods

- [Meteor.call](http://docs.meteor.com/#/full/meteor_call)

## Additional Packages

### ReactiveDict

```javascript
import { reactivedict } from "react-native-meteor";
```

See [documentation](https://atmospherejs.com/meteor/reactive-dict).

### Accounts

```javascript
import Meteor, { Accounts } from "react-native-meteor";
```

- [Accounts.createUser](http://docs.meteor.com/#/full/accounts_createuser)
- [Accounts.changePassword](http://docs.meteor.com/#/full/accounts_forgotpassword)
- [Accounts.forgotPassword](http://docs.meteor.com/#/full/accounts_changepassword)
- [Accounts.resetPassword](http://docs.meteor.com/#/full/accounts_resetpassword)
- [Accounts.onLogin](http://docs.meteor.com/#/full/accounts_onlogin)
- [Accounts.onLoginFailure](http://docs.meteor.com/#/full/accounts_onloginfailure)
- [Meteor.user()](http://docs.meteor.com/#/full/meteor_user)
- [Meteor.userId()](http://docs.meteor.com/#/full/meteor_userid)
- [Meteor.loggingIn()](http://docs.meteor.com/#/full/meteor_loggingin)
- [Meteor.loginWithPassword](http://docs.meteor.com/#/full/meteor_loginwithpassword) (Please note that user is auto-resigned in - like in Meteor Web applications - thanks to React Native AsyncStorage.)
- [Meteor.logout](http://docs.meteor.com/#/full/meteor_logout)
- [Meteor.logoutOtherClients](http://docs.meteor.com/#/full/meteor_logoutotherclients)

### React Meteor Data

```javascript
import { withTracker } from "react-native-meteor";
```

See [documentation](https://atmospherejs.com/meteor/react-meteor-data).

---

# Contribution

Pull Requests and issues reported are welcome! :)
