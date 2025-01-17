[![Build Status](https://api.travis-ci.org/fluttercommunity/redux.dart.svg?branch=master)](https://travis-ci.org/fluttercommunity/redux.dart)
[![codecov](https://codecov.io/gh/fluttercommunity/redux.dart/branch/master/graph/badge.svg)](https://codecov.io/gh/fluttercommunity/redux.dart)
[![Flutter Community: redux](https://fluttercommunity.dev/_github/header/redux)](https://github.com/fluttercommunity/community)


[Redux](https://redux.js.org/) for Dart using generics for typed State. It includes a rich ecosystem of [Docs](#docs), [Middleware](#middleware), [Dev Tools](#dev-tools) and can be combined with Flutter using the [flutter_redux](https://pub.dartlang.org/packages/flutter_redux) package.

## Docs

  * [Motivation and Principles](https://github.com/fluttercommunity/redux.dart/blob/master/doc/why.md) - Learn why Redux might make sense for your app and the principles behind it.
  * [Basics](https://github.com/fluttercommunity/redux.dart/blob/master/doc/basics.md) - Introduction to the core concepts in Redux
  * [Combining Reducers](https://github.com/fluttercommunity/redux.dart/blob/master/doc/combine_reducers.md) - `combineReducers` works a bit differently in Dart than it does in JS. Learn why, and how to combine reducers in a type-safe way!
  * [Async with Middleware](https://github.com/fluttercommunity/redux.dart/blob/master/doc/async.md) - Learn how to make async calls, such as to a web service or local database, with Redux.
  * [Use Selectors to Query Data in the Store](https://github.com/brianegan/reselect_dart/blob/master/README.md) - Learn how to write functions to provide consistent access to data in your App State.
  * [API Documentation](https://www.dartdocs.org/documentation/redux/latest) - Rich documentation included in the source code and generated by DartDoc.

## Flutter Examples

To integrate Redux.dart with Flutter, please use the [flutter_redux](https://pub.dartlang.org/packages/flutter_redux) package. 

### Beginner

  * [Counter example from flutter_redux](https://gitlab.com/brianegan/flutter_redux/tree/master/example) library - A counter example demonstrating the basics of Redux + Flutter
  
### Intermediate 

  * [flutter_redux_todo_list](https://github.com/xqwzts/flutter-redux-todo-list) - A flutter_redux todo list example by [xqwzts](https://github.com/xqwzts).
  
### Advanced

  * [inKino App](https://github.com/roughike/inKino) - A cross platform movie and showtime browser for Finnkino cinemas built with Redux. Lots of testing included as well! 
  * [flutter_architecture_samples](https://gitlab.com/brianegan/flutter_architecture_samples/tree/master/example/redux) - A Todo List App with local storage and multiple screens. Includes a README describing how to combine Redux with Flutter effectively.
  * [Weight Tracking App](https://github.com/MSzalek-Mobile/weight_tracker/) - Demonstrates how to combine Redux with Firebase to build a Flutter app by [MarcinusX](https://github.com/MarcinusX).
       * [Reduxing Flutter Firebase App](https://marcinszalek.pl/flutter/reduxing-flutter/) - Article describing how the author combined Redux with Firebase.
       * [Deleting entry and undoing deletion in snackbar](https://marcinszalek.pl/flutter/deleting-entry-with-undo/) - Article showing how to create "Undo" actions with Redux & Firebase. 

## Middleware

  * [redux_logging](https://pub.dartlang.org/packages/redux_logging) - Connects a [Logger](https://pub.dartlang.org/packages/logging) to a Store, and can print out actions as they're dispatched to your console.
  * [redux_thunk](https://pub.dartlang.org/packages/redux_thunk) - Allows you to dispatch functions that perform async work as actions.
  * [redux_future](https://pub.dartlang.org/packages/redux_future) - For handling Dart Futures that are dispatched as Actions.
  * [redux_epics](https://pub.dartlang.org/packages/redux_epics) - Middleware that allows you to work with Dart Streams of Actions to perform async work.

## Dev Tools

The [redux_dev_tools](https://pub.dartlang.org/packages/redux_dev_tools) library allows you to create a `DevToolsStore` during dev mode in place of a normal Redux `Store`. 

This `DevToolsStore` will act exactly like a normal `Store` at first. However, it will also allow you to travel back and forth throughout the States of your app or recompute the State of your app by replaying all actions through your reducers. This works perfectly with Hot Reloading!

You can combine the `DevToolsStore` with your own UI to travel in time, or use one of the existing options for the platform you're working with:

  * *Flutter* - [flutter_redux_dev_tools](https://pub.dartlang.org/packages/flutter_redux_dev_tools)
  * *Web* - [angular_redux_dev_tools](https://pub.dartlang.org/packages/angular_redux_dev_tools)
  * *Remote* - [Redux Remote Devtools](https://pub.dartlang.org/packages/angular_redux_dev_tools). Connect your Redux.dart store to [Remote DevTools](http://extension.remotedev.io/) and debug in a web browser
  
## Additional Utilities

  * [reselect](https://pub.dartlang.org/packages/reselect) - Efficiently derive data from your Redux Store with memoized functions.
  * [redux_persist](https://github.com/Cretezy/redux_persist) - Persist Redux State, works for Web and Flutter

## Usage

```dart
import 'package:redux/redux.dart';

// Create typed actions. You will dispatch these in order to
// update the state of your application.
enum Actions {
  increment,
  decrement,
}

// Create a Reducer. A reducer is a pure function that takes the 
// current State (int) and the Action that was dispatched. It should
// combine the two into a new state without mutating the state passed
// in! After the state is updated, the store will emit the update to 
// the `onChange` stream.
// 
// Because reducers are pure functions, they should not perform any 
// side-effects, such as making an HTTP request or logging messages
// to a console. For that, use Middleware.
int counterReducer(int state, action) {
  if (action == Actions.increment) {
    return state + 1;
  } else if (action == Actions.decrement) {
    return state - 1;
  }
  
  return state;
}

// A piece of middleware that will log all actions with a timestamp
// to your console!
// 
// Note, this is just an example of how to write your own Middleware.
// See the redux_logging package on pub for a pre-built logging 
// middleware.
loggingMiddleware(Store<int> store, action, NextDispatcher next) {
  print('${new DateTime.now()}: $action');

  next(action);
}

main() {
  // Create the store with our Reducer and Middleware
  final store = new Store<int>(
    counterReducer, 
    initialState: 0, 
    middleware: [loggingMiddleware],
  );

  // Render our State right away
  render(store.state);
  
  // Listen to store changes, and re-render when the state is updated
  store.onChange.listen(render);

  // Attach a click handler to a button. When clicked, the `INCREMENT` action
  // will be dispatched. It will then run through the reducer, updating the 
  // state.
  //
  // After the state changes, the html will be re-rendered by our `onChange`
  // listener above. 
  querySelector('#increment').onClick.listen((_) {
    store.dispatch(Actions.increment);
  });
}

render(int state) {
  querySelector('#value').innerHtml = '${state}';
}
```
