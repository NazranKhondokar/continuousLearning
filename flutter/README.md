# Flutter Best Practices for App Development

Learn the best practices for Flutter developers to improve code quality, readability, maintainability, and productivity. Let’s get started:

#### 1. Make the Build Function Pure
The `build` method should remain pure, as external factors can trigger a widget rebuild. Some examples include:

- Route pop/push
- Screen resize due to keyboard appearance or orientation change
- Parent widget recreating its child
- Changes in an `InheritedWidget` the widget depends on (`of(context)` pattern)

**Avoid:**
```dart
@override
Widget build(BuildContext context) {
  return FutureBuilder(
    future: httpCall(),
    builder: (context, snapshot) {
      // create some layout here
    },
  );
}
```

**Recommended:**
```dart
class Example extends StatefulWidget {
  @override
  _ExampleState createState() => _ExampleState();
}

class _ExampleState extends State<Example> {
  Future<int> future;

  @override
  void initState() {
    future = repository.httpCall();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: future,
      builder: (context, snapshot) {
        // create some layout here
      },
    );
  }
}
```

#### 2. Understand Constraints in Flutter
A key Flutter layout rule: **Constraints go down, sizes go up, and the parent sets the position.**

- A widget’s size must fit within its parent’s constraints.
- Widgets notify their parent of their size within the original constraints.

For instance, a child widget cannot decide its own size independently. It must adhere to the parent’s constraints.

#### 3. Smart Use of Operators

**Cascades Operator:**
```dart
// Do
var path = Path()
  ..lineTo(0, size.height)
  ..lineTo(size.width, size.height)
  ..lineTo(size.width, 0)
  ..close();

// Avoid
var path = Path();
path.lineTo(0, size.height);
path.lineTo(size.width, size.height);
path.lineTo(size.width, 0);
path.close();
```

**Spread Collections:**
```dart
// Do
var y = [4, 5, 6];
var x = [1, 2, ...y];

// Avoid
var y = [4, 5, 6];
var x = [1, 2];
x.addAll(y);
```

**Null-safe (??) and Null-aware (?.) Operators:**
```dart
// Do
v = a ?? b;
v = a?.b;

// Avoid
v = a == null ? b : a;
v = a == null ? null : a.b;
```

**Use “Is” Instead of “As”:**
```dart
// Do
if (item is Animal) {
  item.name = 'Lion';
}

// Avoid
(item as Animal).name = 'Lion';
```

#### 4. Use Streams Only When Needed
Streams are powerful but should be used cautiously to avoid excessive memory or CPU usage. Consider alternatives like `ChangeNotifier` for reactive UI or the `Bloc` library for advanced use cases.

Always call `Sink.close()` to stop the associated `StreamController` in `StatefulWidget.dispose`:
```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  MyBloc bloc;

  @override
  void dispose() {
    bloc.bar.close();
    bloc.foo.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

#### 5. Write Tests for Critical Functionality
Automated tests save time and effort, especially when targeting multiple platforms. Aim for at least unit and widget tests for critical functionality. Full code coverage is ideal but not always practical.

#### 6. Use Raw Strings
```dart
// Do
var s = r'This is a demo string and $';

// Avoid
var s = 'This is a demo string \ and $';
```

#### 7. Use Relative Imports
Avoid mixing relative and absolute imports to prevent confusion.
```dart
// Do
import '../../themes/style.dart';

// Avoid
import 'package:myapp/themes/style.dart';
```

#### 8. Prefer SizedBox Over Container for Placeholders
Use `SizedBox` instead of `Container` for placeholders to improve performance:
```dart
// Do
return _isNotLoaded ? SizedBox() : YourAppropriateWidget();

// Avoid
return _isNotLoaded ? Container() : YourAppropriateWidget();
```

#### 9. Use Log Instead of Print
For detailed logging, use `dart:developer log`:
```dart
// Do
log('data: $data');

// Avoid
print('data: $data');
```

#### 10. Avoid Explicit Null Initialization
```dart
// Do
int _item;

// Avoid
int _item = null;
```

#### 11. Use `const` Keyword Whenever Possible
Using `const` for widgets reduces the garbage collector’s workload and improves performance:
```dart
// Do
const SizedBox(height: 16);

// Avoid
SizedBox(height: 16);
```

#### 12. Additional Tips
- Wrap root widgets in a `SafeArea`.
- Use `final`/`const` for class variables when possible.
- Avoid unnecessary commented code.
- Use private variables and methods (`_variableName`).
- Separate classes for colors, text styles, dimensions, constants, etc.
- Avoid global variables and functions.
- Check `dart analyze` and follow recommendations.
- Use meaningful names for variables instead of “magic numbers.”

**Example:**
```dart
// Do
final _frameIconSize = 13.0;
SvgPicture.asset(
  Images.frameWhite,
  height: _frameIconSize,
  width: _frameIconSize,
);

// Avoid
SvgPicture.asset(
  Images.frameWhite,
  height: 13.0,
  width: 13.0,
);
```

---

### Principles of Clean Code  

#### Naming Conventions  
**Variables and Methods**  
Use camelCase for variables, functions, and parameter names. Descriptive names improve readability.  

```dart
// Bad
var x = 10;
void foo() {}

// Good
var itemCount = 10;
void fetchData() {}
```  

**Classes**  
Use PascalCase for class names. Names should reflect the entity or concept.  

```dart
// Bad
class Foo {}

// Good
class HomeViewModel {}
```  

**File Name**  
Use snake_case for file names.  

---

#### Consistent Formatting  

**Indentation**  
Use consistent indentation (typically 2 spaces for Dart).  

**Style Guide**  
Use tools like `flutter_lints` to enforce coding standards.  

**Braces**  
Always use braces for conditional statements and loops.  

```dart
// Bad
if (condition)
  doSomething();

// Good
if (condition) {
  doSomething();
}
```  

---

#### Simplicity  
Simplify logic by breaking down complex logic into smaller, manageable functions.  

```dart
// Bad
bool isEven(int number) {
  if (number % 2 == 0) {
    return true;
  } else {
    return false;
  }
}

// Good
bool isEven(int number) => number % 2 == 0;
```  

#### Organizing Your Project  

**Data Layer**  
Handles data retrieval and storage.  

```
lib/
├── data/
│   ├── models/
│   └── data sources/
```  

**Domain Layer**  
Contains business logic and use cases.  

```
├── domain/
│   ├── provider/
│   └── repositories/
```  

**Presentation Layer**  
Manages UI and user interaction.  

```
└── presentation/
    ├── screens/
    └── widgets/
```  
 

## Performance Optimization

### Avoid Unnecessary Widget Builds

#### Use const constructors
Whenever possible, use const constructors for immutable widgets.
```dart
// Bad
return Text('Hello World');

// Good
return const Text('Hello World');
```

#### Use keys
Keys help Flutter determine whether widgets should be reused or recreated.
```dart
// Bad
return ListView(
  children: items.map((item) => Text(item)).toList(),
);

// Good
return ListView(
  children: items.map((item) => Text(item, key: Key(item))).toList(),
);
```

#### Use ListView.builder
For long lists, use ListView.builder to lazily build list items.
```dart
// Bad
return ListView(
  children: [
    Text('Item 1'),
    Text('Item 2'),
    // More items...
  ],
);

// Good
return ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Text(items[index]);
  },
);
```

#### Cache Images
Use CachedNetworkImage to cache images for better performance.
```dart
CachedNetworkImage(
  imageUrl: "https://example.com/image.jpg",
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
);
```

#### Lazy Loading
Implement lazy loading for long lists or paginated data.
```dart
LazyLoadScrollView(
  onEndOfPage: () => fetchMoreData(),
  child: ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return ListTile(title: Text(items[index]));
    },
  ),
);
```

### Network Optimization

#### Choose the Right HTTP Client
Use http package for basic requests or Dio for advanced features like interceptors, retries, and timeouts.
```dart
// Example with Dio
Dio dio = Dio();
Future fetchData() async {
  try {
    var response = await dio.get('https://example.com/data');
    print(response.data);
  } catch (e) {
    print(e);
  }
}
```

#### Implement Caching
- Use shared_preferences or hive for device-level caching.
- Use API-level caching to reduce the number of calls to the server.

#### Data Compression
- Compress data before sending to reduce network usage.
- Consider using gzip compression.

#### Error Handling
- Implement robust error handling to gracefully handle network failures.
- Provide informative error messages to the user.

#### Progress Indicators
Display progress indicators during network requests to improve user experience.

#### API Optimization
Minimize data transfer by sending or receiving only necessary data.

### Image Optimization

#### Use CachedNetworkImage
Efficiently loads and caches images.

#### Compress Images
Reduce image size without compromising quality.

#### Use Placeholders
Display placeholders while images are loading.

#### Lazy Loading
Load images only when needed to improve performance.

### Data Fetching Optimization

#### Pagination
Load data in chunks to improve performance and reduce initial load time.

#### Infinite Scrolling
Load more data as the user scrolls.

#### Selective Field Selection
Fetch only required fields from the server or database.

### Memory Management

#### Dispose Controllers
Always dispose of controllers like TextEditingController, AnimationController, etc., to free up resources.
```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final TextEditingController _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
}
```

#### Memory Leaks
- Use tools like Flutter DevTools to identify memory leaks.
- Pay attention to global variables and static fields.
