---
title: Flutter Project Setup
date: 2021-01-20T14:02:26.515Z
image: https://images.freeimages.com/images/large-previews/0c0/blocks-1420958.jpg
tags:
  - Flutter
  - Development
  - Software
  - Mobile
  - Applications
  - ExerciseAppSeries
draft: true
---
## Building Blocks

Any piece of software will have a start point, which when you're new is a difficult thing to work out. What do I need to build this app? What libraries do that for me? Should I use MVVM or MVP or another architecture? What's the benefits of X over Y? These questions can cause people to struggle on the first hurdle, when really we just want a project setup with an architecture baked in to follow.

In my opinion, using MVP or MVVM doesn't matter, using one HTTP library over another doesn't matter, and so forth. It's better to have *consistent* architecture over trying to make sure it's *perfect*.

> [Perfect is the enemy of Good](https://en.wikipedia.org/wiki/Perfect_is_the_enemy_of_good)

Constantly striving for perfection will lead you to spending too much time when Good will get the job done just fine. As you get better, your "Good" will become "Great" from experience, and what was hard will become easy with repitition.

## How I start Flutter Applications
With this in mind, and with the few Flutter Applications I've built, there's a set architecture that I implement immediately after starting a new project. This uses:
- MVVM pattern
- Service layer with interfaces
- ui/core separation
- Locator for Dependency Injection
- Localizey Intl library for i18n
- Static Router class for navigation throughout the application

Anything after this is optional and may include:
- Dio for HTTP client (you'll probably _need_ a http client of some sort, I like Die, you can use whatever you want)
- Firebase
- Flavours for different types of build (dev/staging/prod)
- A websocket library
- etc

## The Application
### Premise
For this application we'll be building an Exercise application which will link to a web server of some sort, pull lists of featured workouts, exercises and instructional videos for a user to follow along at home and practice in their own time. I do Capoeira, so my application will be tailored towards teaching/learning Capoeira at home.

### First Things First
To start I've set up a [Github repository](https://github.com/Contract16/project-suinguera), added a project to it and created some cards to work from which cover a large part of the application. Currently we don't require designs as we're creating a foundation for the app to be built upon.

For this portion of the series we will work on:
- Locator
- i18n
- MVVM with a `BaseView` and `BaseViewModel`
- Package Structure
- Navigator

Once these are in place we have a good foundation to build out our application from there and add new screens/features.

To start, load up Android Studio or VSCode and create a new Flutter project. We'll be modifying the standard counter app and using it to make sure our changes work before we remove the default UI in favour of our own. If you want, feel free to remove all the comments that come with the starter project to help see the code.

### Locator
For this we'll be using [GetIt](https://pub.dev/packages/get_it). From the docs - _"This is a simple Service Locator for Dart and Flutter projects with some additional goodies highly inspired by Splat. It can be used instead of InheritedWidget or Provider to access objects e.g. from your UI."_

We'll be using this to create all of our dependencies for is, generate them as a Singleton or in a Factory, and inject them into any other classes that require them. This gives us a single place where our dependencies are defined so we will never `new` up a Service, ViewModel, API class or anything else.

Update your `pubspec.yaml` in the root of the project to include the `GetIt` library as follows:
```yaml
dependencies:
  ...other dependencies..
  get_it: ^4.0.2
```

Run the `pub get` command to download the dependency and let's set this up.

Next, create a new file `locator.dart` inside the `lib` folder, create a reference to the GetIt instance and declare a function to setup the locator.

```dart
import 'package:get_it/get_it.dart';

var locator = GetIt.instance;

Future<void> setupLocator() async {

}

```

Currently there are no dependencies to setup so this empty function is fine. I set this up as a `Future<void> async` function at the start to allow me to add lazy dependencies later if I want to, but you can just convert the function later if you need and leave the signature as `setupLocator() { ... }` for now. This for me is a personal preference.

Now we're going to need to call the `setupLocator()` method, so update your `main.dart` class to call the function before running the app. We'll also need to convert the `main()` function to a `Future<void> async` function so we can call the setup function correctly.

```dart
Future<void> main() async {
  await setupLocator();
  runApp(MyApp());
}
```

And that's it for the locator! A few lines and it's ready to be used throughout the app! Feel free to commit this and let's set up our MVVM.

### Base Classes
On the whole be wary of Base Classes. A lot of the time in the wild they become a [God Class](https://en.wikipedia.org/wiki/God_object). If you ever define a Base Class then be sure you know exactly what it's for and why it's there, and then don't add to it unless it's _absolutely neccessary_. With proper architecture this shouldn't be a concern, you should always favour [Composition over Inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance) if possible.

#### BaseViewModel
This is a simple class which all of our app's `ViewModel` classes will extend. It will contain a state and a function to update the state, and itself will extend a `ChangeNotifier` so that any UI which is using the `ViewModel` to observe a variable can update itself when any changes occer.

First we'll set up the package structure as follows:
```
-lib
--ui
---page
----base
```

Once this package structure is defined, create 3 new `dart` files inside of the `base` package, these will be `base_view.dart`, `base_view_model.dart`, and `view_state.dart`.

We'll fill in `view_state.dart` first with the following:
```dart
enum ViewState {
  Idle,
  Busy
}
```

This enum will be used inside of the `BaseViewModel` to allow the UI to know if the ViewModel is currently busy and as such the UI will be able to react accordingly and show a loading state or other UI if required.

Next let's make the `BaseViewModel`. Inside of `base_view_model.dart` declare a `BaseViewModel` class and make it extend 