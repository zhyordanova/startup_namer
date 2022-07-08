**My First Flutter App**

***1. Introduction***

Flutter is Google's UI toolkit for building beautiful, natively compiled applications for mobile, web, and desktop from a single codebase. Flutter works with existing code, is used by developers and organizations around the world, and is free and open source.
In this codelab, you'll create a simple mobile Flutter app. If you're familiar with object-oriented code and basic programming concepts—such as variables, loops, and conditionals—then you can complete the codelab. You don't need previous experience with Dart, mobile, desktop, or web programming.

***2. What you'll build***

You`ll implement a simple app that generates proposed names for a startup company. The user can select and unselect names, saving the best ones. The code lazily generates 10 names at a time. As the user scrolls, more names are generated. There is no limit to how far a user can scroll.
The following animated GIF shows how the app works at the completion of part:

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt1/img/e3624172607d5551.gif)

***3. Create the starter Flutter app***

Create a simple, templated Flutter app. Create a Flutter project called startup_namer as follows:
$ flutter create startup_namer
$ cd startup_namer
You'll mostly edit lib/main.dart, where the Dart code lives.

***4. Use an external package***

In this step, you'll start using an open-source package named english_words, which contains a few thousand of the most-used English words, plus some utility functions.
    * Add the english_words package as a dependency of this app:

       $ flutter pub add english_words
        
   * In lib/main.dart, import the new package:

            import 'package:english_words/english_words.dart';
            import 'package:flutter/material.dart';



•	This example creates a Material app. Material is a visual-design language that's standard on mobile and the web. Flutter offers a rich set of Material widgets.\
•	The app extends StatelessWidget, which makes the app itself a widget. In Flutter, almost everything is a widget, including alignment, padding, and layout.\
•	The Scaffold widget, from the Material library, provides a default app bar, a title, and a body property that holds the widget tree for the home screen. The widget subtree can be quite complex.\
•	A widget's main job is to provide a build method that describes how to display the widget in terms of other, lower-level widgets.\
•	The body for this example consists of a Center widget containing a Text child widget. The Center widget aligns its widget subtree to the center of the screen.

            void main() {
              runApp(const MyApp());
            }
            
            class MyApp extends StatelessWidget {
              const MyApp({Key? key}) : super(key: key);
            
              @override
              Widget build(BuildContext context) {
                final wordPair = WordPair.random();
                return MaterialApp(
                  title: 'Startup Name Generator',
                  home: Scaffold(
                    appBar: AppBar(
                      title: const Text('Startup Name Generator'),
                    ),
                    body: Center(                          
                      child: Text(wordPair.asPascalCase),  
                    ),
                  ),
                );
              }
            }
If the app is running, hot reload   to update the running app. (From the command line, you can enter r to hot reload.) Each time you click hot reload or save the project, you should see a different word pair, chosen at random, in the running app. That's because the word pairing is generated inside the build method, which runs each time the MaterialApp requires rendering, or when toggling the Platform in the Flutter Inspector.

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt1/img/35f55fd09f2ff2d9.png)

***5. Add a stateful widget***

Stateless widgets are immutable, meaning that their properties can't change—all values are final.
Stateful widgets maintain state that might change during the lifetime of the widget. Implementing a stateful widget requires at least two classes, a StatefulWidget that creates an instance of a State class. The StatefulWidget object is, itself, immutable and can be thrown away and regenerated, but the State object persists over the lifetime of the widget.
In this step, you'll add a stateful widget, RandomWords, which creates its State class, _RandomWordsState. You'll then use RandomWords as a child inside the existing MyApp stateless widget.
* Create boilerplate code for a stateful widget.

It can go anywhere in the file outside of MyApp, but the solution places it at the bottom of the file. In lib/main.dart, position your cursor after all of the code, enter Return a couple times to start on a fresh line. In your IDE, start typing stful. The editor asks if you want to create a Stateful widget. Press Return to accept. The boilerplate code for two classes appears, and the cursor is positioned for you to enter the name of your stateless widget.

* Enter RandomWords as the name of your widget.
Once you've entered RandomWords as the name of the stateful widget, the IDE automatically updates the accompanying State class, naming it _RandomWordsState. By default, the name of the State class is prefixed with an underscore. Prefixing an identifier with an underscore enforces privacy in the Dart language and is a recommended best practice for State objects.\
The IDE also automatically updates the state class to extend State<RandomWords>, indicating that you're using a generic State class specialized for use with RandomWords. Most of the app's logic resides here⁠—it maintains the state for the RandomWords widget. This class saves the list of generated word pairs, which grows infinitely as the user scrolls and, in part 2 of this lab, favorites word pairs as the user adds or removes them from the list by toggling the heart icon.
Both classes now look as follows:

            class RandomWords extends StatefulWidget {
              const RandomWords({ Key? key }) : super(key: key);

              @override
              State<RandomWords> createState() => _RandomWordsState();
            }

            class _RandomWordsState extends State<RandomWords> {
              @override
              Widget build(BuildContext context) {
                return Container();
              }
            }
            
* Update the build() method in _RandomWordsState.
            
            class _RandomWordsState extends State<RandomWords> {
              @override                                  
              Widget build(BuildContext context) {
                final wordPair = WordPair.random();      
                return Text(wordPair.asPascalCase);      
              }                                         
            }

* Remove the word-generation code from MyApp by making the following changes:

            class MyApp extends StatelessWidget {
              @override
              Widget build(BuildContext context) {

                return MaterialApp(
                  title: 'Startup Name Generator',
                  home: Scaffold(
                    appBar: AppBar(
                      title: Text('Startup Name Generator'),
                    ),
                    body: const Center(                    
                      child: RandomWords(),        
                    ),
                  ),
                );
              }
            }

***6. Create an infinite scrolling ListView***

In this step, you'll expand _RandomWordsState to generate and display a list of word pairings. As the user scrolls, the list (displayed in a ListView widget) grows infinitely. The builder factory constructor in ListView allows you to lazily build a list view on demand.

* Add some state variables to the _RandomWordsState class.

Add a _suggestions list for saving suggested word pairings. Also, add a _biggerFont variable for making the font size larger.

            class _RandomWordsState extends State<RandomWords> {
              final _suggestions = <WordPair>[];               
              final _biggerFont = const TextStyle(fontSize: 18); 
              ...
            }
Next, you'll update the build method of the _RandomWordsState class.
The ListView class provides a builder property, itemBuilder, that's a factory builder and callback function specified as an anonymous function. Two parameters are passed to the function—the BuildContext and the row index, i. The index begins at 0 and increments each time the function is called, once for every suggested word pairing. This model allows the suggestion list to continue growing as the user scrolls.

* Update the build method for _RandomWordsState.

Change it to use _buildSuggestions(), rather than directly calling the word-generation library.

            @override
              Widget build(BuildContext context) {
                return ListView.builder(
                  padding: const EdgeInsets.all(16.0),
                  itemBuilder: (context, i) {
                    if (i.isOdd) return const Divider();

                    final index = i ~/ 2;
                    if (index >= _suggestions.length) {
                      _suggestions.addAll(generateWordPairs().take(10));
                    }
                    return ListTile(
                      title: Text(
                        _suggestions[index].asPascalCase,
                        style: _biggerFont,
                      ),
                    );
                  },
                );
              }
              
The itemBuilder callback is called once per suggested word pairing, and places each suggestion into a ListTile row. For even rows, the function adds a ListTile row for the word pairing. For odd rows, the function adds a Divider widget to visually separate the entries. Note that the divider may be difficult to see on smaller devices.
Note: The syntax `i ~/ 2` divides `i` by `2` and returns an integer result. For example, the list `[1,2,3,4,5]`  becomes `[0,1,1,2,2]`. This calculates the actual number of word pairings in the ListView, minus the divider widgets.

* Update the build method for MyApp, changing the title in two places.

            @override
              Widget build(BuildContext context) {
                return MaterialApp(
                  title: 'Startup Name Generator',
                  home: Scaffold(
                    appBar: AppBar(
                      title: const Text('Startup Name Generator'),
                    ),
                    body: const Center(
                      child: RandomWords(),
                    ),
                  ),
                );
              }

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt1/img/d71590c8c3c2e6f8.png)

***7. Add icons to the list***

In this step, you'll add heart icons to each row. In the next step, you'll make them tappable and save the favorites.

* Add a _saved Set to _RandomWordsState. This Set stores the word pairings that the user favorited. Set is preferred to List because a properly implemented Set doesn't allow duplicate entries.

            class _RandomWordsState extends State<RandomWords> {
              final _suggestions = <WordPair>[];
              final _saved = <WordPair>{};     
              final _biggerFont = TextStyle(fontSize: 18);
              ...
            }
            
* In the build function, add an alreadySaved check to ensure that a word pairing has not already been added to favorites.

            final alreadySaved = _saved.contains(_suggestions[index]);
            Add the icons after the text, as shown below:
            return ListTile(
              title: Text(
                _suggestions[index].asPascalCase,
                style: _biggerFont,
              ),
              trailing: Icon( 
                alreadySaved ? Icons.favorite : Icons.favorite_border,
                color: alreadySaved ? Colors.red : null,
                semanticLabel: alreadySaved ? 'Remove from saved' : 'Save',
              ),             
            );

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt2/img/eca56f73115e3550.png)

***8. Add interactivity***

In this step, you'll make the heart icons tappable. When the user taps an entry in the list, toggling its favorited state, that word pairing is added or removed from a set of saved favorites.\
To do that, you'll modify the _buildRow function. If a word entry has already been added to favorites, tapping it again removes it from favorites. When a tile has been tapped, the function calls setState() to notify the framework that state has changed.

* Add onTap to the build method, as shown below:

            return ListTile(
              title: Text(
                _suggestions[index].asPascalCase,
                style: _biggerFont,
              ),
              trailing: Icon(
                alreadySaved ? Icons.favorite : Icons.favorite_border,
                color: alreadySaved ? Colors.red : null,
                semanticLabel: alreadySaved ? 'Remove from saved' : 'Save',
              ),
              onTap: () {         
                setState(() {
                  if (alreadySaved) {
                    _saved.remove(_suggestions[index]);
                  } else {
                    _saved.add(_suggestions[index]);
                  }
                });                
              },
            );
 
![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt2/img/9d55495e865ccdaf.png)

***9. Navigate to a new screen*** 

In Flutter, the Navigator manages a stack containing the app's routes. Pushing a route onto the Navigator's stack updates the display to that route. Popping a route from the Navigator's stack returns the display to the previous route.\
Next, you'll add a list icon to the AppBar in the build method for _RandomWordsState. When the user clicks the list icon, a new route that contains the saved favorites is pushed to the Navigator, displaying the icon.

* Remove the Scaffold from the MyApp widget:

            class MyApp extends StatelessWidget {
              const MyApp({Key? key}) : super(key: key);

              @override
              Widget build(BuildContext context) {
                return const MaterialApp(          // MODIFY with const
                  title: 'Startup Name Generator',
                  home: RandomWords(),             // REMOVE Scaffold
                );
              }
            }
            
The Scaffold is going to reappear in the _RandomWordsState. This will enable you to add a IconButton to the Scaffold's AppBar that interacts with the state of the application.\
Wrap the ListView.builder with the Scaffold from MyApp, and add an IconButton to the AppBar's actions parameter, in the build method of the _RandomWordsState class:

            class _RandomWordsState extends State<RandomWords> {
              ...
              @override
              Widget build(BuildContext context) {
                return Scaffold(
                  appBar: AppBar(  
                    title: const Text('Startup Name Generator'),
                    actions: [
                      IconButton(
                        icon: const Icon(Icons.list),
                        onPressed: _pushSaved,
                        tooltip: 'Saved Suggestions',
                      ),
                    ],
                  ),               
                  body: ListView.builder(  
                  ...
                  ),                       
                );
              }
              ...
            }

Tip: Some widget properties take a single widget (child), and other properties, such as action, take an array of widgets (children), as indicated by the square brackets ([]).

* Add a _pushSaved() function to the _RandomWordsState class.

            void _pushSaved() {
              }

Next, you'll build a route and push it to the Navigator's stack. That action changes the screen to display the new route. The content for the new page is built in MaterialPageRoute's builder property in an anonymous function.

* Call Navigator.push, as shown below, which pushes the route to the Navigator's stack. The IDE will complain about invalid code, but you will fix that in the next section.

            void _pushSaved() {
              Navigator.of(context).push(
              );
            }

Next, you'll add the MaterialPageRoute and its builder. For now, add the code that generates the ListTile rows. The divideTiles() method of ListTile adds horizontal spacing between each ListTile. The divided variable holds the final rows converted to a list by the convenience function, toList().

* Add the code, as shown in the following code snippet:

            void _pushSaved() {
                Navigator.of(context).push(
                  MaterialPageRoute<void>(
                    builder: (context) {
                      final tiles = _saved.map(
                        (pair) {
                          return ListTile(
                            title: Text(
                              pair.asPascalCase,
                              style: _biggerFont,
                            ),
                          );
                        },
                      );
                      final divided = tiles.isNotEmpty
                          ? ListTile.divideTiles(
                              context: context,
                              tiles: tiles,
                            ).toList()
                          : <Widget>[];

                      return Scaffold(
                        appBar: AppBar(
                          title: const Text('Saved Suggestions'),
                        ),
                        body: ListView(children: divided),
                      );
                    },
                  ), 
                );
              }
              
The builder property returns a Scaffold containing the app bar for the new route named SavedSuggestions. The body of the new route consists of a ListView containing the ListTiles rows. Each row is separated by a divider.

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt2/img/6e06cd704eec2745.png) ![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt2/img/a9f9050220d86dad.png)

10. Change the UI using themes

You can easily change an app's theme by configuring the ThemeData class. The app uses the default theme, but you'll change the app's primary color to white.

* Change the color in the MyApp class:

            class MyApp extends StatelessWidget {
              const MyApp({Key? key}) : super(key: key);

              @override
              Widget build(BuildContext context) {
                return MaterialApp(          // Remove the const from here
                  title: 'Startup Name Generator',
                  theme: ThemeData(          
                    appBarTheme: const AppBarTheme(
                      backgroundColor: Colors.white,
                      foregroundColor: Colors.black,
                    ),
                  ),                     
                  home: const RandomWords(),   // And add the const back here.
                );
              }
            }

![](https://codelabs.developers.google.com/static/codelabs/first-flutter-app-pt2/img/2ae336b38fe4c72a.png)

***Well done!***

You wrote an interactive Flutter app that runs on iOS, Android, Windows, Linux, macOS, and web by doing the following:\
•	Writing Dart code\
•	Using hot reload for a faster development cycle\
•	Implementing a stateful widget, adding interactivity to your app\
•	Creating a route and adding logic for moving between the home route and the new route\
•	Learning about changing the look of your app's UI using themes
