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
- Package Structure
- MVVM with a `BaseView` and `BaseViewModel`

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

Next let's make the `BaseViewModel`. Inside of `base_view_model.dart` declare a `BaseViewModel` class and make it extend `ChangeNotifier`. This will give us access to `notifyListeners()` which will tell the UI that something inside the ViewModel has changed.

```dart
class BaseViewModel extends ChangeNotifier {

}
```

Inside this class we are going to declare a private `ViewState`, provide a getter for this, and create the function to update this.

```dart
class BaseViewModel extends ChangeNotifier {
  ViewState _viewState = ViewState.Idle;

  ViewState get viewState => _viewState;

  void setState(ViewState viewState) {
    _viewState = viewState;
    notifyListeners();
  }
}
```

Now every time we update any variable inside any of our future ViewModels we can simply call `setState(...)` and the UI will be notified that the ViewModel has changes to react to.

#### BaseView
We have our `BaseViewModel` set up, so we can now create a `BaseView` to utilise this. Our `BaseView` will be a `StatefulWidget` and will listen for changes inside of a `BaseViewModel`. To start we want to create the `BaseView` widget inside of `base_view.dart`.

```
import 'package:flutter/material.dart';

class BaseView extends StatefulWidget{
  @override
  _BaseViewState createState() => _BaseViewState();
}

class _BaseViewState extends State<BaseView> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    throw UnimplementedError();
  }
}

```

From this we can expand on the implementation by giving the `BaseView` a generic type which is also `BaseViewModel` by modifying the class declaration line as follows:

```dart
class BaseView<T extends BaseViewModel> extends StatefulWidget {
```

This will allow us to use our locator to provide our ViewModel internal to the `BaseView` later on. We'll need to update the rest of the class to match and pass in the generic `T` to the View's State.

```dart
class BaseView<T extends BaseViewModel> extends StatefulWidget {
  @override
  _BaseViewState<T> createState() => _BaseViewState<T>();
}

class _BaseViewState<T extends BaseViewModel> extends State<BaseView<T>> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    throw UnimplementedError();
  }
}
```

So now we have a `_BaseViewState` of type `BaseViewModel`, and we want to get a `ViewModel` to use. With our locator, this is as simple as asking for one as a class level variable inside of the `_BaseViewState` class
```
T _viewModel = locator<T>();
```

As long as we've registered factory for a ViewModel of type `T` with the locator then we'll receive a ViewModel for use here.

The next step is to allow the view to react to changes, for this we'll need to import the `provider` library into our project and use a `ChangeNotifierProvider` which listens for any changes inside of a given `ChangeNotifier`, such as our `BaseViewModel` is.

Open your `pubspec.yaml` and add the following dependency:
```yaml
dependencies:
  ...other dependencies...
  provider: ^4.3.3
```

Run the `pub get` command again and then inside of the `build` function of `_BaseViewState` we're going to return a `ChangeNotifierProvider`.

```
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<T>(
      create: (context) => viewModel,
      child: Consumer<T>(
        builder: null,
      ),
    );
  }
```

As you can see, a `ChangeNotifierProvider` requires a `create` function, for which we'll provide our `viewModel` and we'll use a `Consumer` widget to listen for changes which propogate from the `ChangeNotifierProvider`. This is the bit which will refresh when the ViewModel notifies any listeners of changes. We also have a `builder` function which is where we inject our normal UI, but we need a way to inject our UI into this method.

In our `BaseView` class we'll add a field with a builder function which will mimic the builder function of a `Consumer`, which is as follows:
```dart
final Widget Function(BuildContext context, T value, Widget child) builder;
```

We can have exactly the same function inside the `BaseView` and access this from our `_BaseViewState`.
```dart
class BaseView<T extends BaseViewModel> extends StatefulWidget {
  final Widget Function(BuildContext context, T value, Widget child) builder;

  const BaseView({Key key, @required this.builder}) : super(key: key);

  @override
  _BaseViewState<T> createState() => _BaseViewState<T>();
}

class _BaseViewState<T extends BaseViewModel> extends State<BaseView<T>> {
  T viewModel = locator<T>();

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<T>(
      create: (context) => viewModel,
      child: Consumer<T>(
        builder: widget.builder,
      ),
    );
  }
}
```

And now our `BaseView` is setup and ready to go. What's left is updating our home page to use this.

### MVVM In Practice
We have our `locator`, we have our `BaseView` and `BaseViewModel`, and now we want to use this in our home page. First, let's do a bit of restructuring. We want to create a new package inside of `lib.ui.page` called `home`, so you'll have the following package structure:
```
-lib
--ui
---page
----base
----home
```

Inside of `home` create 2 new dart files, `home_page.dart` and `home_view_model.dart`. Then go into `main.dart` and move the `MyHomePage` and `_MyHomePageState` classes into `home_page.dart`. Fix any imports and run the code to make sure the app still runs. We haven't used anything new yet, so if something doesn't work here then it's likely occured when moving the code.

Next step we're going to move all of the counter logic out of the Home Page and into the ViewModel, the Home Page will only be used for display. Open `home_view_model.dart` and create a new `HomeViewModel`
```dart
class HomeViewModel extends BaseViewModel {

}
```

After this let's move the counter variable and `_incrementCounter` function into here.
```dart
class HomeViewModel extends BaseViewModel {
  int _counter = 0;

  int get counter => _counter;

  void incrementCounter() {
    setState(ViewState.Busy);
    _counter++;
    setState(ViewState.Idle);
  }
}
```

Remember that whenever we want to do anything which effects UI inside of a ViewModel we should call `setState(...)` so that any listeners are told about potential changes.

Once this is done we can use our `BaseView` inside of the Home Page to access our `HomeViewModel`.
```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return BaseView<HomeViewModel>(
      builder: (context, viewModel, child) => Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Text(
                'You have pushed the button this many times:',
              ),
              Text(
                '${viewModel.counter}',
                style: Theme.of(context).textTheme.headline4,
              ),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: viewModel.incrementCounter,
          tooltip: 'Increment',
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

Here we're creating a new `BaseView` widget of type `HomeViewModel`. This means that the new `BaseView` widget will ask the `locator` for a `HomeViewModel` to use and will grant access to this in the `builder` which we're using to present our screen. Currently the `locator` doesn't know about a `HomeViewModel` so if you run this code now you'll get an error, so let's fix that.

To register any class in the locator is quite simple, we go to `locator.dart` and add the registration to the `setupLocator()` function. Typically Services are registered as Singleton objects, and ViewModels are registered as Factory objects. Each time you ask for a ViewModel now the locator will create a new one for you. Here's how the `setupLocator()` function should look now:
```dart
Future setupLocator() async {
  locator.registerFactory(() => HomeViewModel());
}
```

Run the app again and if you've followed the steps all the way through you should now have a working Counter app following MVVM, dependency injection, with a separation between View and ViewModel. This will become a lot more powerful later on when we're building the application, but for now it's nice to see it all working as intended :)

Part 2 soon which will involve internationalisation (i18n) and using a Navigator to move around to new pages in your application.