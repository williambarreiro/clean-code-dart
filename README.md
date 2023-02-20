# Clean Code in Dart
This repository is an adaptation of [ryanmcdermott/clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript) for Dart language.

## Table of Contents
1. [Introduction](#introduction)
2. [Variables](#variables)
3. [Functions](#functions)
4. [Objects and Data Structures](#objects-and-data-structures)
5. [Classes](#classes)
6. [SOLID](#solid)
7. [Testing](#testing)
8. [Concurrency](#concurrency)
9. [Error Handling](#error-handling)
10. [Formatting](#formatting)
11. [Comments](#comments)
12. [Translation](#translation)

## Introduction

![Humorous image of software quality estimation as a count of how many expletives
you shout when reading code](https://www.osnews.com/images/comics/wtfm.jpg)

Software engineering principles, from Robert C. Martin's book
[_Clean Code_](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for Dart. This is not a style guide. It's a guide to producing readable, reusable, and refactorable software in Dart.

Not every principle herein has to be strictly followed, and even fewer will be
universally agreed upon. These are guidelines and nothing more, but they are
ones codified over many years of collective experience by the authors of
_Clean Code_.

Our craft of software engineering is just a bit over 50 years old, and we are
still learning a lot. When software architecture is as old as architecture
itself, maybe then we will have harder rules to follow. For now, let these
guidelines serve as a touchstone by which to assess the quality of the
Dart code that you and your team produce.

One more thing: knowing these won't immediately make you a better software
developer, and working with them for many years doesn't mean you won't make
mistakes. Every piece of code starts as a first draft, like wet clay getting
shaped into its final form. Finally, we chisel away the imperfections when
we review it with our peers. Don't beat yourself up for first drafts that need
improvement. Beat up the code instead!

## **Variables**

### Use meaningful and pronounceable variable names

**Bad:**

```dart
final yyyymmdstr = DateFormat('yyyy/MM/dd').format(DateTime.now());
```

**Good:**

```dart
final currentDate = DateFormat('yyyy/MM/dd').format(DateTime.now());
```

**[⬆ back to top](#table-of-contents)**

### Use the same vocabulary for the same type of variable

**Bad:**

```dart
getUserInfo();
getClientData();
getCustomerRecord();
```

**Good:**

```dart
getUser();
```

**[⬆ back to top](#table-of-contents)**

### Use searchable names

We will read more code than we will ever write. It's important that the code we
do write is readable and searchable. By _not_ naming variables that end up
being meaningful for understanding our program, we hurt our readers.
Make your names searchable.

**Bad:**

```dart
// What the heck is 32 for?
Future.delayed(Duration(minutes: 32), launch);
```

**Good:**

```dart
// Declare them as const if the value is known at compile time;
// Declare as final if the variable is assigned just once;
// Use lowerCamelCase;
// setupTimeInMinutes is int, because the type is inferred.
const setupTimeInMinutes = 32;

Future.delayed(Duration(minutes: setupTimeInMinutes), launch);
```

**[⬆ back to top](#table-of-contents)**

### Use explanatory variables

**Bad:**

```dart
const address = <String>['One Infinite Loop', 'Cupertino', '95014'];
saveCityZipCode(address[1], address[2]);
```

**Good:**

```dart
const address = <String>['One Infinite Loop', 'Cupertino', '95014'];
final city = address[1];
final zipCode = address[2];
saveCityZipCode(city, zipCode);
```

**[⬆ back to top](#table-of-contents)**

### Avoid Mental Mapping

Explicit is better than implicit.

**Bad:**

```dart
const locations = <String>['Austin', 'New York', 'San Francisco'];
locations.forEach((l) {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  // Wait, what is `l` for again?
  dispatch(l);
});
```

**Good:**

```dart
const locations = <String>['Austin', 'New York', 'San Francisco'];
locations.forEach((location) {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  dispatch(location);
});
```

**[⬆ back to top](#table-of-contents)**

### Don't add unneeded context

If your class/object name tells you something, don't repeat that in your
variable name.

**Bad:**

```dart
final car = Car(
  carMake: 'Honda',
  carModel: 'Accord',
  carColor: 'Blue',
);

void paintCar(Car car, String color) {
  car.carColor = color;
}
```

**Good:**

```dart
final car = Car(
  make: 'Honda',
  model: 'Accord',
  color: 'Blue',
);

void paintCar(Car car, String color) {
  car.color = color;
}
```

**[⬆ back to top](#table-of-contents)**

### When possible, use default parameters instead of short circuiting or conditionals

Default parameters are often cleaner than short circuiting, however they must be const, so you cannot use them on every case.

**Bad:**

```dart
void createMicrobrewery({String? name}) {
  final breweryName = name ?? 'Hipster Brew Co.';
  // ...
}
```

**Good:**

```dart
void createMicrobrewery({String breweryName = 'Hipster Brew Co.'}) {
  // ...
}
```

**[⬆ back to top](#table-of-contents)**

## **Functions**

### Function arguments (2 or fewer ideally)

Limiting the amount of function parameters is incredibly important because it
makes testing your function easier. Having more than three leads to a
combinatorial explosion where you have to test tons of different cases with
each separate argument.

One or two arguments is the ideal case, and three should be avoided if possible.
Anything more than that should be consolidated. Usually, if you have
more than two arguments then your function is trying to do too much. In cases
where it's not, most of the time a higher-level object will suffice as an
argument.

To make it obvious what properties the function expects, you can use named
parameters. They have a few advantages:

1. When someone looks at the function signature, it's immediately clear what
   properties are being used.
2. Linters can warn you about unused properties, if they are `required`.

**Bad:**

```dart
Menu getMenu(String title, String body, String buttonText, bool cancellable) {
  // ...
}
```

**Good:**

```dart
Menu getMenu({
  required String title,
  required String body,
  required String buttonText,
  required bool cancellable,
}) {
  // ...
}

final menu = getMenu(
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true,
);
```

**[⬆ back to top](#table-of-contents)**

### Functions should do one thing

This is by far the most important rule in software engineering. When functions
do more than one thing, they are harder to compose, test, and reason about.
When you can isolate a function to just one action, it can be refactored
easily and your code will read much cleaner. If you take nothing else away from
this guide other than this, you'll be ahead of many developers.

**Bad:**

```dart
void emailClients(List<Client> clients) {
  for(final client in clients) {
    final clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  }
}
```

**Good:**

```dart
void emailActiveClients(List<Client> clients) {
  clients
    .where(isActiveClient)
    .forEach(email);
}

bool isActiveClient(Client client) {
  final clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

**[⬆ back to top](#table-of-contents)**

### Function names should say what they do

**Bad:**

```dart
void addToDate(DateTime date, int months) {
  // ...
}

final currentDate = DateTime.now();

// It's hard to tell from the function name what is added
addToDate(currentDate, 1);
```

**Good:**

```dart
void addMonthsToDate(int months, DateTime date) {
  // ...
}

final currentDate = DateTime.now();
addMonthsToDate(1, currentDate);
```

**[⬆ back to top](#table-of-contents)**

### Functions should only be one level of abstraction

When you have more than one level of abstraction your function is usually
doing too much. Splitting up functions leads to reusability and easier
testing.

**Bad:**

```dart
void parseBetterAlternative(String code) {
  const regexes = [
    // ...
  ];

  final statements = code.split(' ');
  final tokens = [];
  for (final regex in regexes) {
    for (final statement in statements) {
      tokens.add( /* ... */ );
    }
  }

  final ast = <Node>[];
  for (final token in tokens) {
    ast.add( /* ... */ );
  }

  for (final node in ast) {
    // parse...
  }
}
```

**Good:**

```dart
List<String> tokenize(String code) {
  const regexes = [
    // ...
  ];

  final statements = code.split(' ');
  final tokens = <String>[];
  for (final regex in regexes) {
    for (final statement in statements) {
      tokens.add( /* ... */ );
    }
  }

  return tokens;
}

List<Node> lexer(List<String> tokens) {
  final ast = <Node>[];
  for (final token in tokens) {
    ast.add( /* ... */ );
  }
  
  return ast;
}

void parseBetterAlternative(String code) {
  final tokens = tokenize(code);
  final ast = lexer(tokens);
  for (final node in ast) {
    // parse...
  }
}
```

**[⬆ back to top](#table-of-contents)**

### Remove duplicate code

Do your absolute best to avoid duplicate code. Duplicate code is bad because it
means that there's more than one place to alter something if you need to change
some logic.

Imagine if you run a restaurant and you keep track of your inventory: all your
tomatoes, onions, garlic, spices, etc. If you have multiple lists that
you keep this on, then all have to be updated when you serve a dish with
tomatoes in them. If you only have one list, there's only one place to update!

Oftentimes you have duplicate code because you have two or more slightly
different things, that share a lot in common, but their differences force you
to have two or more separate functions that do much of the same things. Removing
duplicate code means creating an abstraction that can handle this set of
different things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the
SOLID principles laid out in the _Classes_ section. Bad abstractions can be
worse than duplicate code, so be careful! Having said this, if you can make
a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself
updating multiple places anytime you want to change one thing.

**Bad:**

```dart
Widget buildDeveloperCard(Developer developer) {
  return CustomCard(
    expectedSalary: developer.calculateExpectedSalary(),
    experience: developer.getExperience(),
    projectsLink: developer.getGithubLink(),
  );
}

Widget buildManagerCard(Manager manager) {
  return CustomCard(
    expectedSalary: manager.calculateExpectedSalary(),
    experience: manager.getExperience(),
    projectsLink: manager.getMBAProjects(),
  );
}
```

**Good:**

```dart
Widget buildEmployeeCard(Employee employee) {
  String projectsLink;

  if (employee is Manager) {
    projectsLink = manager.getMBAProjects();
  } else if (employee is Developer) {
    projectsLink = developer.getGithubLink();
  }

  return CustomCard(
    expectedSalary: employee.calculateExpectedSalary(),
    experience: employee.getExperience(),
    projectsLink: projectsLink,
  );
}
```

**[⬆ back to top](#table-of-contents)**

### Don't use flags as function parameters

Flags tell your user that this function does more than one thing. Functions should do one thing. Split out your functions if they are following different code paths based on a boolean.

**Bad:**

```dart
void createFile(String name, bool temp) {
  if (temp) {
    File('./temp/${name}').create();
  } else {
    File(name).create();
  }
}
```

**Good:**

```dart
void createFile(String name) {
  File(name).create();
}

void createTempFile(String name) {
  File('./temp/${name}').create();
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid Side Effects (part 1)

A function produces a side effect if it does anything other than take a value in
and return another value or values. A side effect could be writing to a file,
modifying some global variable, or accidentally wiring all your money to a
stranger.

Now, you do need to have side effects in a program on occasion. Like the previous
example, you might need to write to a file. What you want to do is to
centralize where you are doing this. Don't have several functions and classes
that write to a particular file. Have one service that does it. One and only one.

The main point is to avoid common pitfalls like sharing state between objects
without any structure, using mutable data types that can be written to by anything,
and not centralizing where your side effects occur. If you can do this, you will
be happier than the vast majority of other programmers.

**Bad:**

```dart
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
dynamic name = 'Ryan McDermott';

void splitIntoFirstAndLastName() {
  name = name.split(' ');
}

splitIntoFirstAndLastName();

print(name); // ['Ryan', 'McDermott'];
```

**Good:**

```dart
List<String> splitIntoFirstAndLastName(name) {
  return name.split(' ');
}

final name = 'Ryan McDermott';
final newName = splitIntoFirstAndLastName(name);

print(name); // 'Ryan McDermott';
print(newName); // ['Ryan', 'McDermott'];
```

**[⬆ back to top](#table-of-contents)**

### Avoid Side Effects (part 2)

In Dart, some values are unchangeable (immutable) and some are changeable 
(mutable). Objects and arrays are two kinds of mutable values so it's important 
to handle them carefully when they're passed as parameters to a function. A 
Dart function can change an object's properties or alter the contents of 
an array which could easily cause bugs elsewhere.

Suppose there's a function that accepts an array parameter representing a 
shopping cart. If the function makes a change in that shopping cart array - 
by adding an item to purchase, for example - then any other function that 
uses that same `cart` array will be affected by this addition. That may be 
great, however it could also be bad. Let's imagine a bad situation:

The user clicks the "Purchase" button which calls a `purchase` function that
spawns a network request and sends the `cart` array to the server. Because
of a bad network connection, the `purchase` function has to keep retrying the
request. Now, what if in the meantime the user accidentally clicks an "Add to Cart"
button on an item they don't actually want before the network request begins?
If that happens and the network request begins, then that purchase function
will send the accidentally added item because the `cart` array was modified.

A great solution would be for the `addItemToCart` function to always clone the 
`cart`, edit it, and return the clone. This would ensure that functions that are still
using the old shopping cart wouldn't be affected by the changes.

Two caveats to mention to this approach:

1. There might be cases where you actually want to modify the input object,
   but when you adopt this programming practice you will find that those cases
   are pretty rare. Most things can be refactored to have no side effects!

2. Cloning big objects can be very expensive in terms of performance. Luckily,
   this isn't a big issue in practice because there are
   [great libraries](https://facebook.github.io/immutable-js/) that allow
   this kind of programming approach to be fast and not as memory intensive as
   it would be for you to manually clone objects and arrays.

**Bad:**

```dart
void addItemToCart(List<int> cart, int item) {
  cart.add(item);
} 

final cart = <int>[1, 2];
addItemToCart(cart, 3);

print(cart); // [1, 2, 3]
```

**Good:**

```dart
List<int> addItemToCart(List<int> cart, int item) {
  return [...cart, item];
}

final cart = <int>[1, 2];
final newCart = addItemToCart(cart, 3);

print(cart); // [1, 2]
print(newCart); // [1, 2, 3]
```

**[⬆ back to top](#table-of-contents)**

### Favor functional programming over imperative programming

Dart isn't a functional language in the way that Haskell is, but it has
a functional flavor to it. Functional languages can be cleaner and easier to test.
Favor this style of programming when you can.

**Bad:**

```dart
final programmerOutput = <Programmer>[
  Programmer(name: 'Uncle Bobby', linesOfCode: 500),
  Programmer(name: 'Suzie Q', linesOfCode: 1500),
  Programmer(name: 'Jimmy Gosling', linesOfCode: 150),
  Programmer(name: 'Gracie Hopper', linesOfCode: 1000),
];

var totalOutput = 0;

for (var i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

**Good:**

```dart
final programmerOutput = <Programmer>[
  Programmer(name: 'Uncle Bobby', linesOfCode: 500),
  Programmer(name: 'Suzie Q', linesOfCode: 1500),
  Programmer(name: 'Jimmy Gosling', linesOfCode: 150),
  Programmer(name: 'Gracie Hopper', linesOfCode: 1000),
];

final totalOutput = programmerOutput.fold<int>(
    0, (previousValue, programmer) => previousValue + programmer.linesOfCode);
```

**[⬆ back to top](#table-of-contents)**

### Encapsulate conditionals

**Bad:**

```dart
if (programmer.language == 'dart' && programmer.projectsList.isNotEmpty) {
  // ...
}
```

**Good:**

```dart
bool isValidDartProgrammer(Programmer programmer) {
  return programmer.language == 'dart' && programmer.projectsList.isNotEmpty;
}

if (isValidDartProgrammer(programmer)) {
  // ...
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid negative conditionals

**Bad:**

```dart
bool isFileNotValid(File file) {
  // ...
}

if (!isFileNotValid(file)) {
  // ...
}
```

**Good:**

```dart
bool isFileValid(File file) {
  // ...
}

if (isFileValid(file)) {
  // ...
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid conditionals

This seems like an impossible task. Upon first hearing this, most people say,
"how am I supposed to do anything without an `if` statement?" The answer is that
you can use polymorphism to achieve the same task in many cases. The second
question is usually, "well that's great but why would I want to do that?" The
answer is a previous clean code concept we learned: a function should only do
one thing. When you have classes and functions that have `if` statements, you
are telling your user that your function does more than one thing. Remember,
just do one thing.

**Bad:**

```dart
class Airplane {
  // ...
  double getCruisingAltitude() {
    switch (type) {
      case '777':
        return getMaxAltitude() - getPassengerCount();
      case 'Air Force One':
        return getMaxAltitude();
      case 'Cessna':
        return getMaxAltitude() - getFuelExpenditure();
    }
  }
}
```

**Good:**

```dart
class Airplane {
  // ...
}

class Boeing777 extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude() - getPassengerCount();
  }
}

class AirForceOne extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude();
  }
}

class Cessna extends Airplane {
  // ...
  double getCruisingAltitude() {
    return getMaxAltitude() - getFuelExpenditure();
  }
}
```

**[⬆ back to top](#table-of-contents)**

### Remove dead code

Dead code is just as bad as duplicate code. There's no reason to keep it in
your codebase. If it's not being called, get rid of it! It will still be safe
in your version history if you still need it.

**Bad:**

```dart
Future<void> oldRequest(url) {
  // ...
}

Future<void> newRequest(url) {
  // ...
}

await newRequest();
```

**Good:**

```dart
Future<void> newRequest(url) {
  // ...
}

await newRequest();
```

**[⬆ back to top](#table-of-contents)**

## **Objects and Data Structures**

### Use getters and setters only when necessary

Unlike other languages, in Dart it is recommended to use getters and setters only when there is some logic before using the attribute. If you just want to get or edit the attribute, don't use them.

**Bad:**

```dart
class BankAccount {
  // "_" configure as private
  int _balance;

  int get balance => _balance;

  set balance(int amount) => _balance = amount;

  BankAccount({
    int balance = 0,
  }) : _balance = balance;
}

final account = BankAccount();
account.balance = 100;
```

**Good:**

```dart
class BankAccount {
  int balance;
  // ...

  BankAccount({
    this.balance = 0,
    // ...
  });
}

final account = BankAccount();
account.balance = 100;
```

**[⬆ back to top](#table-of-contents)**

### Use private methods and attributes when necessary

If a method or attribute has to be accessed only within the class, it must be private.

**Bad:**

```dart
class Employee {
  String name;

  Employee({required this.name});
}

final employee = Employee(name: 'John Doe');
print(employee.name); // John Doe
employee.name = 'Uncle Bob';
print(employee.name); // Uncle Bob
```

**Good:**

```dart
class Employee {
  String _name;

  Employee({required String name}) : _name = name;
}

final employee = Employee(name: 'John Doe');
print(employee.name); // Can't access outside the class.
```

**[⬆ back to top](#table-of-contents)**

## **Classes**

### Use method chaining (cascade notation)

It allows your code to be expressive, and less verbose. For that reason, I say, 
use method chaining and take a look at how clean your code will be.

**Bad:**

```dart
class Car {
  String make;
  String model;
  String color;

  Car({
    required this.make,
    required this.model,
    required this.color,
  });

  save() => print('$make, $model, $color');
}

final car = Car(make: 'Ford', model: 'F-150', color: 'red');
car.color = 'pink';
car.save();
```

**Good:**

```dart
class Car {
  String make;
  String model;
  String color;

  Car({
    required this.make,
    required this.model,
    required this.color,
  });

  save() => print('$make, $model, $color');
}

final car = Car(make: 'Ford', model: 'F-150', color: 'red')
  ..color = 'pink'
  ..save();
```

**[⬆ back to top](#table-of-contents)**

### Prefer composition over inheritance

As stated famously in [_Design Patterns_](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four,
you should prefer composition over inheritance where you can. There are lots of
good reasons to use inheritance and lots of good reasons to use composition.
The main point for this maxim is that if your mind instinctively goes for
inheritance, try to think if composition could model your problem better. In some
cases it can.

You might be wondering then, "when should I use inheritance?" It
depends on your problem at hand, but this is a decent list of when inheritance
makes more sense than composition:

1. Your inheritance represents an "is-a" relationship and not a "has-a"
   relationship (Human->Animal vs. User->UserDetails).
2. You can reuse code from the base classes (Humans can move like all animals).
3. You want to make global changes to derived classes by changing a base class.
   (Change the caloric expenditure of all animals when they move).

**Bad:**

```dart
class Employee {
  String name;
  String email;

  Employee({
    required this.name,
    required this.email,
  });

  // ...
}

// Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee
class EmployeeTaxData extends Employee {
  String ssn;
  double salary;

  EmployeeTaxData({
    required this.ssn,
    required this.salary,
    required super.name,
    required super.email,
  });

  // ...
}
```

**Good:**

```dart
class EmployeeTaxData {
  String ssn;
  double salary;

  EmployeeTaxData({
    required this.ssn,
    required this.salary,
  });

  // ...
}

class Employee {
  String name;
  String email;
  EmployeeTaxData? taxData;

  Employee({
    required this.name,
    required this.email,
  });

  void setTaxData(String ssn, double salary) {
    taxData = EmployeeTaxData(ssn: ssn, salary: salary);
  }

  // ...
}
```

**[⬆ back to top](#table-of-contents)**

## **SOLID**

### Single Responsibility Principle (SRP)

As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimizing the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify
a piece of it, it can be difficult to understand how that will affect other
dependent modules in your codebase.

**Bad:**

```dart
class UserSettings {
  String user;
  
  UserSettings({
    required this.user,
  });

  void changeSettings(Settings settings) {
    if (verifyCredentials()) {
      // ...
    }
  }

  bool verifyCredentials() {
    // ...
  }
}
```

**Good:**

```dart
class UserAuth {
  String user;

  UserAuth({
    required this.user,
  });

  bool verifyCredentials() {
    // ...
  }
}

class UserSettings {
  String user;
  UserAuth auth;

  UserSettings({
    required this.user,
  }) : auth = UserAuth(user: user);

  void changeSettings(Settings settings) {
    if (auth.verifyCredentials()) {
      // ...
    }
  }
}
```

**[⬆ back to top](#table-of-contents)**

### Open/Closed Principle (OCP)

As stated by Bertrand Meyer, "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

**Bad:**

```dart
double getArea(Shape shape) {
  if (shape is Circle) {
    return getCircleArea(shape);
  } else if (shape is Square) {
    return getSquareArea(shape);
  }
}

double getCircleArea(Shape shape) {
  // ...
}

double getSquareArea(Shape shape) {
  // ...
}
```

**Good:**

```dart
abstract class Shape {
  double getArea();
}

class Circle extends Shape {
  @override
  double getArea() {
    // ...
  }
}

class Square extends Shape {
  @override
  double getArea() {
    // ...
  }
}

// ...
final area = shape.getArea();
```

**[⬆ back to top](#table-of-contents)**

### Liskov Substitution Principle (LSP)

This is a scary term for a very simple concept. It's formally defined as "If S
is a subtype of T, then objects of type T may be replaced with objects of type S
(i.e., objects of type S may substitute objects of type T) without altering any
of the desirable properties of that program (correctness, task performed,
etc.)." That's an even scarier definition.

The best explanation for this is if you have a parent class and a child class,
then the base class and child class can be used interchangeably without getting
incorrect results. This might still be confusing, so let's take a look at the
classic Square-Rectangle example. Mathematically, a square is a rectangle, but
if you model it using the "is-a" relationship via inheritance, you quickly
get into trouble.

**Bad:**

```dart
class Rectangle {
  double width;
  double height;

  Rectangle({
    this.width = 0,
    this.height = 0,
  });

  // setWidth e setHeight used just for example
  void setWidth(double value) => width = value;

  void setHeight(double value) => height = value;

  double getArea() {
    return width * height;
  }
}

class Square extends Rectangle {
  Square({
    super.width = 0,
    super.height = 0,
  });

  @override
  void setWidth(double value) {
    width = value;
    height = value;
  }

  @override
  void setHeight(double value) {
    width = value;
    height = value;
  }
}

final rectangles = [Rectangle(), Rectangle(), Square()];

for (final rectangle in rectangles) {
  rectangle.setWidth(4);
  rectangle.setHeight(5);

  final area = rectangle.getArea();
  print(area); // BAD: Returns 25 for Square. Should be 20.
}
```

**Good:**

```dart
abstract class Shape {
  double getArea();
}

class Rectangle extends Shape {
  double width;
  double height;

  Rectangle({
    required this.width,
    required this.height,
  });

  @override
  double getArea() {
    return width * height;
  }
}

class Square extends Shape {
  double length;

  Square({
    required this.length,
  });

  @override
  double getArea() {
    return length * length;
  }
}

final rectangles = [
  Rectangle(width: 4, height: 5),
  Rectangle(width: 4, height: 5),
  Square(length: 4),
];

for (final rectangle in rectangles) {
  final area = rectangle.getArea();
  print(area); // Show the correct values: 20, 20, 16.
}
```

**[⬆ back to top](#table-of-contents)**

### Interface Segregation Principle (ISP)

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use". You should always create more specific interfaces instead of creating just a generic interface. In other words, if your class, that implements an interface, uses the famous `throw UnimplementedError()`, it is probably not respecting the principle.

**Bad:**

```dart
abstract class Book {
  int getNumberOfPages();
  void download();
}

class EBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  String download() {
    // ...
  }
}

class PhysicalBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  void download() {
    throw UnimplementedError(); // Physical book doesn't download.
  }
}
```

**Good:**

```dart
abstract class Book {
  int getNumberOfPages();
}

abstract class DownloadableBook {
  void download();
}

class EBook implements Book, DownloadableBook {
  @override
  int getNumberOfPages() {
    // ...
  }

  @override
  void download() {
    // ...
  }
}

class PhysicalBook implements Book {
  @override
  int getNumberOfPages() {
    // ...
  }
}
```

**[⬆ back to top](#table-of-contents)**

### Dependency Inversion Principle (DIP)

This principle states two essential things:

1. High-level modules should not depend on low-level modules. Both should
   depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
   abstractions.

You've probably seen an implementation of this principle in the form of Dependency
Injection (DI). While they are not identical concepts, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

**Bad:**

```dart
class InventoryRequester {
  void requestItem(item) {
    // ...
  }
}

class InventoryTracker {
  final requester = InventoryRequester(); // InventoryTracker depends on low-level module.
  List<String> items;

  InventoryTracker({
    required this.items,
  });

  void requestItems() {
    for (var item in items) {
      requester.requestItem(item);
    }
  }
}

final inventoryTracker = InventoryTracker(items: ['apples', 'bananas']);
inventoryTracker.requestItems();
```

**Good:**

```dart
class InventoryTracker {
  List<String> items;
  InventoryRequester requester;

  InventoryTracker({
    required this.items,
    required this.requester,
  });

  void requestItems() {
    for (var item in items) {
      requester.requestItem(item);
    }
  }
}

abstract class InventoryRequester {
  void requestItem(item);
}

class InventoryRequesterV1 implements InventoryRequester {
  @override
  void requestItem(item) {
    // ...
  }
}

class InventoryRequesterV2 implements InventoryRequester {
  @override
  void requestItem(item) {
    // ...
  }
}

// By constructing our dependencies externally and injecting them, we can easily
// substitute our request module for a fancy new one.
final inventoryTracker = InventoryTracker(
  items: ['apples', 'bananas'],
  requester: InventoryRequesterV2(),
);
inventoryTracker.requestItems();
```

**[⬆ back to top](#table-of-contents)**

## **Testing**

Testing is more important than shipping. If you have no tests or an
inadequate amount, then every time you ship code you won't be sure that you
didn't break anything. Deciding on what constitutes an adequate amount is up
to your team, but having 100% coverage (all statements and branches) is how
you achieve very high confidence and developer peace of mind.

Always write tests for every new feature/module you introduce. If your preferred
method is Test Driven Development (TDD), that is great, but the main point is to just
make sure you are reaching your coverage goals before launching any feature,
or refactoring an existing one.

### Single concept per test

**Bad:**

```dart
import 'package:test/test.dart';

test('String', () {
  var string = 'foo,bar,baz';
  expect(string.split(','), equals(['foo', 'bar', 'baz']));

  string = '  foo ';
  expect(string.trim(), equals('foo'));
});
```

**Good:**

```dart
import 'package:test/test.dart';

group('String', () {
  test('.split() splits the string on the delimiter', () {
    const string = 'foo,bar,baz';
    expect(string.split(','), equals(['foo', 'bar', 'baz']));
  });

  test('.trim() removes surrounding whitespace', () {
    const string = '  foo ';
    expect(string.trim(), equals('foo'));
  });
});
```

**[⬆ back to top](#table-of-contents)**

## **Concurrency**

### Use async/await instead of then

Using async/await makes your code simpler and easier to understand.

**Bad:**

```dart
final albumTitle = await client
    .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'))
    .then((response) {
  // ...
  return title;
});
```

**Good:**

```dart
Future<String> getAlbumTitle() async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  // ...

  return title;
}
```

**[⬆ back to top](#table-of-contents)**

## **Error Handling**

Thrown errors are a good thing! They mean the runtime has successfully
identified when something in your program has gone wrong and it's letting
you know by stopping function execution on the current stack, killing the
process, and notifying you in the console with a stack trace.

### Don't ignore caught errors

Doing nothing with a caught error doesn't give you the ability to ever fix
or react to said error. Logging the error to the console (`log`)
isn't much better as often times it can get lost in a sea of things printed
to the console. If you wrap any bit of code in a `try/catch` it means you
think an error may occur there and therefore you should have a plan,
or create a code path, for when it occurs.

**Bad:**

```dart
try {
  functionThatMightThrow();
} catch (error) {
  print(error);
}
```

**Good:**

```dart
try {
  functionThatMightThrow();
} on Exception catch (e, s) {
  // Option 1:
  log('Error description...', error: e, stackTrace: s);
  // Option 2:
  notifyUserOfError(e, s);
  // Option 3:
  reportErrorToService(e, s);
}
```

### Don't ignore Future errors

Using await within try/catch is way better than future/then. But,
if you want to use future/then, remember to handle the errors.

**Bad:**

```dart
functionThatMightThrow().then((value) {
  // ...
}).onError((e, s) {
  print(e);
});
```

**Good:**

```dart
functionThatMightThrow().then((value) {
  // ...
}).onError((e, s) {
  // Option 1:
  log('Error description...', error: e, stackTrace: s);
  // Option 2:
  notifyUserOfError(e, s);
  // Option 3:
  reportErrorToService(e, s);
});
```

**[⬆ back to top](#table-of-contents)**

## **Formatting**

Formatting is subjective. Like many rules herein, there is no hard and fast
rule that you must follow. The main point is DO NOT ARGUE over formatting.
I recommend you to read [Effective Dart](https://dart.dev/guides/language/effective-dart/style),
there are several rules to be followed, but nothing is mandatory.

### Use the correct capitalization

**Bad:**

```dart
const DAYS_IN_WEEK = 7;

const Bands = ['AC/DC', 'Led Zeppelin', 'The Beatles'];

void restore_database() {}

class animal {}

typedef predicate<T> = bool Function(T value);
```

**Good:**

```dart
// lowerCamelCase for constant names
const daysInWeek = 7;
const bands = ['AC/DC', 'Led Zeppelin', 'The Beatles'];

// lowerCamelCase for functions
void restoreDatabase() {}

// UpperCamelCase for classes, enum types, typedefs, and type parameters
class Animal {}
typedef Predicate<T> = bool Function(T value);
```

**[⬆ back to top](#table-of-contents)**

### Function callers and callees should be close

If a function calls another, keep those functions vertically close in the source
file. Ideally, keep the caller right above the callee. We tend to read code from
top-to-bottom, like a newspaper. Because of this, make your code read that way.

**Bad:**

```dart
class Smartphone {
  // ...

  String getOS() {
    // ...
  }

  void showPlatform() {
    final os = getOS();
    final chipset = getChipset();
    // ...
  }

  String getResolution() {
    // ...
  }

  void showSpecifications() {
    showPlatform();
    showDisplay();
  }

  String getChipset() {
    // ...
  }

  void showDisplay() {
    final resolution = getResolution();
    // ...
  }
}
```

**Good:**

```dart
class Smartphone {
  // ...

  void showSpecifications() {
    showPlatform();
    showDisplay();
  }

  void showPlatform() {
    final os = getOS();
    final chipset = getChipset();
    // ...
  }

  String getOS() {
    // ...
  }

  String getChipset() {
    // ...
  }

  void showDisplay() {
    final resolution = getResolution();
    // ...
  }

  String getResolution() {
    // ...
  }
}
```

**[⬆ back to top](#table-of-contents)**

## **Comments**

### Only comment things that have business logic complexity.

Comments are an apology, not a requirement. Good code _mostly_ documents itself.

**Bad:**

```dart
List<String> getCitiesNames(List<String> cities) {
  // Cities names list
  final citiesNames = <String>[];

  // Loop through every city
  for (final city in cities) {
    // Gets only the string before the comma
    final filteredCityName = city.split(',')[0];

    // Add the filtered city name
    citiesNames.add(filteredCityName);
  }

  // Returns the cities names list
  return citiesNames;
}
```

**Good:**

```dart
List<String> getCitiesNames(List<String> cities) {
  final citiesNames = <String>[];

  for (final city in cities) {
    // Gets only the string before the comma
    final filteredCityName = city.split(',')[0];

    citiesNames.add(filteredCityName);
  }

  return citiesNames;
}
```

**[⬆ back to top](#table-of-contents)**

### Don't leave commented out code in your codebase

Version control exists for a reason. Leave old code in your history.

**Bad:**

```dart
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

**Good:**

```dart
doStuff();
```

**[⬆ back to top](#table-of-contents)**

### Don't have journal comments

Remember, use version control! There's no need for dead code, commented code,
and especially journal comments. Use `git log` to get history!

**Bad:**

```dart
/**
 * 2016-12-20: Removidas monads, não entendia elas (RM)
 * 2016-10-01: Melhoria utilizando monads especiais (JP)
 * 2016-02-03: Removido checagem de tipos (LI)
 * 2015-03-14: Adicionada checagem de tipos (JR)
 */
int combine(int a, int b) {
  return a + b;
}
```

**Good:**

```dart
int combine(int a, int b) {
  return a + b;
}
```

**[⬆ back to top](#table-of-contents)**

### Avoid positional markers

They usually just add noise. Let the functions and variable names along with the
proper indentation and formatting give the visual structure to your code.

**Bad:**

```dart
////////////////////////////////////////////////////////////////////////////////
// Programmer Instantiation
////////////////////////////////////////////////////////////////////////////////
final programmer = Programmer(
  name: 'Jack',
  linesOfCode: 500,
);

////////////////////////////////////////////////////////////////////////////////
// startProject implementation
////////////////////////////////////////////////////////////////////////////////
void startProject() {
  // ...
};
```

**Good:**

```dart
final programmer = Programmer(
  name: 'Jack',
  linesOfCode: 500,
);

void startProject() {
  // ...
};
```

**[⬆ back to top](#table-of-contents)**

## Translation

This is also available in other languages:

- ![br](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Brazil.png) **Brazilian Portuguese**: [williambarreiro/clean-code-dart](https://github.com/williambarreiro/clean-code-dart)

**[⬆ back to top](#table-of-contents)**