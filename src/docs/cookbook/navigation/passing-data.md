---
title: 새로운 화면으로 데이터 보내기
prev:
  title: 이전 화면에 데이터 반환하기
  path: /docs/cookbook/navigation/returning-data
next:
  title: 인터넷에서 데이터 받기
  path: /docs/cookbook/networking/fetch-data
js:
  - defer: true
    url: https://dartpad.dev/inject_embed.dart.js
---

종종 새로운 화면으로 단순히 이동하는 것 뿐만 아니라 
데이터를 넘겨주어야 할 때도 있습니다. 
예를 들어, 사용자가 선택한 아이템에 대한 정보를 
같이 넘겨주고 싶은 경우가 있습니다.

Remember: Screens are just widgets.
In this example, create a list of todos.
When a todo is tapped, navigate to a new screen (widget) that
displays information about the todo.
This recipe uses the following steps:

  1. Todo 클래스를 정의합니다.
  2. Todo 리스트를 보여줍니다.
  3. Todo에 대한 상세 정보를 보여줄 수 있는 화면을 생성합니다.
  4. 상세 화면으로 이동하면서 데이터를 전달합니다.

## 1. Todo 클래스를 정의합니다.

우선 Todo를 표현하기 위한 간단한 정보가 필요합니다. 이 예제에서는 두 가지의 데이터를
갖고 있는 클래스를 정의할 것입니다: 제목과 상세 설명

<!-- skip -->
```dart
class Todo {
  final String title;
  final String description;

  Todo(this.title, this.description);
}
```

## 2. Todo 리스트를 보여줍니다.

Second, display a list of todos. In this example, generate
20 todos and show them using a ListView.
For more information on working with lists,
see the [Use lists][] recipe.

### Todo 리스트 생성하기

<!-- skip -->
```dart
final todos = List<Todo>.generate(
  20,
  (i) => Todo(
        'Todo $i',
        'A description of what needs to be done for Todo $i',
      ),
);
```

### ListView를 사용하여 Todo 리스트 보여주기

<!-- skip -->
```dart
ListView.builder(
  itemCount: todos.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(todos[index].title),
    );
  },
);
```

여기까지 그런대로 잘 되었습니다. 이제 20개의 Todo를 생성하여 ListView에 보여줄 것입니다!

## 3. Todo에 대한 상세 정보를 보여줄 수 있는 화면을 생성합니다.

이제 두 번째 화면을 만들겠습니다. 화면의 제목은 Todo의 제목을 포함하며 본문에는
상세 설명을 보여줄 것입니다.

두 번째 화면은 일반적인 `StatelssWidget`이므로, 
생성자 매개변수로 `Todo`를 받을 수 있게 간단한 방법으로 강제하겠습니다. 
그러면 주어진 Todo를 활용하여 UI를 그릴 수 있습니다.

<!-- skip -->
```dart
class DetailScreen extends StatelessWidget {
  // Todo를 들고 있을 필드를 선언합니다.
  final Todo todo;

  // 생성자 매개변수로 Todo를 받도록 강제합니다.
  DetailScreen({Key key, @required this.todo}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // UI를 그리기 위해 Todo를 사용합니다.
    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

## 4. 상세 화면으로 이동하면서 데이터를 전달합니다.

앞서 작성한 `DetailsScreen`으로 화면 전환할 준비가 되었습니다. 
본 예제에서는 사용자가 Todo 리스트 중 하나를 선택했을 때, 
`DetailsScreen`으로 화면 전환하도록 할 것입니다.
그와 동시에 `DetailsScreen`에 Todo를 전달할 것입니다.

To capture the user's tap, write an [`onTap()`][]
callback for the `ListTile` widget. Within the `onTap()` callback,
use the [`Navigator.push()`][] method.

<!-- skip -->
```dart
ListView.builder(
  itemCount: todos.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(todos[index].title),
      // 사용자가 ListTile을 선택하면, DetailScreen으로 이동합니다.
      // DetailScreen을 생성할 뿐만 아니라, 현재 todo를 같이 전달해야
      // 한다는 것을 명심하세요.
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => DetailScreen(todo: todos[index]),
          ),
        );
      },
    );
  },
);
```

### Interactive example

```run-dartpad:theme-light:mode-flutter:run-true:width-100%:height-600px:split-60
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class Todo {
  final String title;
  final String description;

  Todo(this.title, this.description);
}

void main() {
  runApp(MaterialApp(
    title: 'Passing Data',
    home: TodosScreen(
      todos: List.generate(
        20,
        (i) => Todo(
              'Todo $i',
              'A description of what needs to be done for Todo $i',
            ),
      ),
    ),
  ));
}

class TodosScreen extends StatelessWidget {
  final List<Todo> todos;

  TodosScreen({Key key, @required this.todos}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Todos'),
      ),
      body: ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(todos[index].title),
            // 사용자가 ListTile을 선택하면, DetailScreen으로 이동합니다.
            // DetailScreen을 생성할 뿐만 아니라, 현재 todo를 같이 전달해야
            // 한다는 것을 명심하세요.
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => DetailScreen(todo: todos[index]),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  // Todo를 들고 있을 필드를 선언합니다.
  final Todo todo;

  // 생성자는 Todo를 인자로 받습니다.
  DetailScreen({Key key, @required this.todo}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // UI를 그리기 위해 Todo를 사용합니다.
    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

## Alternatively, pass the arguments using RouteSettings

Repeat the first two steps.

### Create a detail screen to extract the arguments

Next, create a detail screen that extracts and displays the title and description from the `Todo`. To access the `Todo`, use the `ModalRoute.of()` method. This method returns the current route with the arguments.

```dart
class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final Todo todo = ModalRoute.of(context).settings.arguments;

    // Use the Todo to create the UI.
    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

### Navigate and pass the arguments to the detail screen

Finally, navigate to the `DetailScreen` when a user taps a `ListTile` widget using `Navigator.push()`. Pass the arguments as part of the `RouteSettings`. The `DetailScreen` extracts these arguments.

```dart
ListView.builder(                                                    
  itemCount: todos.length,                                           
  itemBuilder: (context, index) {                                    
    return ListTile(                                                 
      title: Text(todos[index].title),                               
      // When a user taps the ListTile, navigate to the DetailScreen.
      // Notice that you're not only creating a DetailScreen, you're 
      // also passing the current todo through to it.                
      onTap: () {                                                    
        Navigator.push(                                              
          context,                                                   
          MaterialPageRoute(                                         
            builder: (context) => DetailScreen(),                    
            // Pass the arguments as part of the RouteSettings. The  
            // DetailScreen reads the arguments from these settings. 
            settings: RouteSettings(                                 
              arguments: todos[index],                               
            ),                                                        
          ),                                                          
        );                                                           
      },                                                             
    );                                                                
  },                                                                 
),                                                                                                                           
```

### Complete example

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class Todo {
  final String title;
  final String description;

  Todo(this.title, this.description);
}

void main() {
  runApp(MaterialApp(
    title: 'Passing Data',
    home: TodosScreen(
      todos: List.generate(
        20,
        (i) => Todo(
          'Todo $i',
          'A description of what needs to be done for Todo $i',
        ),
      ),
    ),
  ));
}

class TodosScreen extends StatelessWidget {
  final List<Todo> todos;

  TodosScreen({Key key, @required this.todos}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Todos'),
      ),
      body: ListView.builder(
        itemCount: todos.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(todos[index].title),
            // When a user taps the ListTile, navigate to the DetailScreen.
            // Notice that you're not only creating a DetailScreen, you're
            // also passing the current todo through to it.
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => DetailScreen(),
                  // Pass the arguments as part of the RouteSettings. The
                  // DetailScreen reads the arguments from these settings.
                  settings: RouteSettings(
                    arguments: todos[index],
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final Todo todo = ModalRoute.of(context).settings.arguments;

    // Use the Todo to create the UI.
    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

<noscript>
  <img src="/images/cookbook/passing-data.gif" alt="Passing Data Demo" class="site-mobile-screenshot" />
</noscript>


[`Navigator.push()`]: {{site.api}}/flutter/widgets/Navigator/push.html
[`onTap()`]: {{site.api}}/flutter/material/ListTile/onTap.html
[Use lists]: /docs/cookbook/lists/basic-list
