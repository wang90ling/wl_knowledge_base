# Flutter 学习路线与实战指南（零基础版）

> 适合完全没有 Flutter 基础的同学，从 Dart 入门到独立开发完整项目。
>
> 本文档已整理成更适合打印、收藏和反复查阅的版本。

---

## 目录

1. Flutter 是什么
2. 学习 Flutter 前要先学什么
3. Dart 基础速成
4. Flutter 核心概念
5. 常用组件与布局
6. 页面跳转与表单
7. 网络请求、JSON、状态管理
8. 从零开发一个 Flutter 项目
9. 零基础 30 天学习计划
10. 常见误区与学习建议
11. 总结

---

## 1. Flutter 是什么

Flutter 是 Google 推出的跨平台 UI 框架。你只需要写一套代码，就可以同时开发：

- Android App
- iOS App
- Web
- Windows / macOS / Linux 桌面应用

### Flutter 的特点

- **跨平台**：一套代码多端运行
- **高性能**：直接使用自己的渲染引擎
- **开发快**：支持热重载，改完代码能快速看到效果
- **UI 统一**：容易做出一致、漂亮的界面

Flutter 使用的语言是 **Dart**。所以，学 Flutter 前先学 Dart，是最稳妥的路线。

---

## 2. 学习 Flutter 前要先学什么

如果你是零基础，建议按这个顺序学习：

### 第一层：Dart 基础

先掌握：

- 变量、常量
- 基本数据类型
- 函数
- 类与对象
- 构造函数
- 集合 `List`、`Map`、`Set`
- 异步 `Future`、`async/await`
- 空安全 `null safety`

### 第二层：Flutter 基础

再掌握：

- Widget 思维
- `StatelessWidget` 和 `StatefulWidget`
- 常见布局组件
- 列表、图片、按钮、输入框
- 页面跳转
- 表单与校验

### 第三层：项目开发能力

继续学习：

- 网络请求
- JSON 解析
- 状态管理
- 本地存储
- 主题、动画、性能优化

---

## 3. Dart 基础速成

### 3.1 变量与类型

```dart
void main() {
  String name = 'Flutter';
  int age = 3;
  double score = 99.5;
  bool isReady = true;

  print(name);
  print(age);
  print(score);
  print(isReady);
}
```

常见类型：

- `String`：字符串
- `int`：整数
- `double`：小数
- `bool`：布尔值

### 3.2 `var`、`final`、`const`

```dart
void main() {
  var city = 'Shenzhen';
  final createdAt = DateTime.now();
  const pi = 3.14159;

  print(city);
  print(createdAt);
  print(pi);
}
```

- `var`：自动推断类型
- `final`：只能赋值一次，值运行时确定
- `const`：编译期常量

### 3.3 函数

```dart
int add(int a, int b) {
  return a + b;
}

int multiply(int a, int b) => a * b;

void main() {
  print(add(2, 3));
  print(multiply(4, 5));
}
```

### 3.4 类与对象

```dart
class User {
  String name;
  int age;

  User(this.name, this.age);

  void sayHello() {
    print('Hello, I am $name, age $age');
  }
}

void main() {
  final user = User('Alice', 20);
  user.sayHello();
}
```

### 3.5 异步

```dart
Future<String> fetchData() async {
  await Future.delayed(const Duration(seconds: 2));
  return '数据加载完成';
}

void main() async {
  print('开始');
  final result = await fetchData();
  print(result);
  print('结束');
}
```

异步是 Flutter 项目里非常重要的基础，因为网络请求、文件读取、数据库访问都需要它。

---

## 4. Flutter 核心概念

### 4.1 一切都是 Widget

在 Flutter 里，文本、按钮、图片、布局、页面，都是 Widget。

你可以把 Flutter 理解成：

> 用一个个 Widget 组合出完整界面。

### 4.2 StatelessWidget 与 StatefulWidget

#### StatelessWidget

适合内容不会变化的页面。

```dart
import 'package:flutter/material.dart';

class HelloPage extends StatelessWidget {
  const HelloPage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(
        child: Text('Hello Flutter'),
      ),
    );
  }
}
```

#### StatefulWidget

适合会变化的页面，比如计数器、列表刷新、表单输入。

```dart
import 'package:flutter/material.dart';

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int count = 0;

  void increase() {
    setState(() {
      count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(
        child: Text(
          'Count: $count',
          style: const TextStyle(fontSize: 32),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: increase,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

---

## 5. 常用组件与布局

### 5.1 `Container`

```dart
Container(
  width: 200,
  height: 100,
  padding: const EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
  ),
  child: const Text(
    'Hello Container',
    style: TextStyle(color: Colors.white),
  ),
)
```

### 5.2 `Row` 与 `Column`

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  children: const [
    Text('第一行'),
    Text('第二行'),
    Text('第三行'),
  ],
)
```

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: const [
    Icon(Icons.star),
    Icon(Icons.favorite),
    Icon(Icons.share),
  ],
)
```

### 5.3 `Stack`

```dart
Stack(
  children: [
    Container(width: 200, height: 200, color: Colors.grey),
    const Positioned(
      bottom: 10,
      right: 10,
      child: Text('Overlay'),
    ),
  ],
)
```

### 5.4 `ListView`

```dart
ListView.builder(
  itemCount: 20,
  itemBuilder: (context, index) {
    return ListTile(
      leading: const Icon(Icons.person),
      title: Text('Item $index'),
      subtitle: const Text('This is a list item'),
    );
  },
)
```

### 5.5 常见按钮

- `ElevatedButton`
- `TextButton`
- `OutlinedButton`
- `FloatingActionButton`

---

## 6. 页面跳转与表单

### 6.1 页面跳转

```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailPage()),
);
```

返回上一页：

```dart
Navigator.pop(context);
```

### 6.2 表单输入与校验

```dart
import 'package:flutter/material.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _usernameController = TextEditingController();

  @override
  void dispose() {
    _usernameController.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      print('用户名: ${_usernameController.text}');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _usernameController,
                decoration: const InputDecoration(labelText: '用户名'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入用户名';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                onPressed: _submit,
                child: const Text('提交'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 7. 网络请求、JSON、状态管理

### 7.1 网络请求

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Post {
  final int id;
  final String title;

  Post({required this.id, required this.title});

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'],
      title: json['title'],
    );
  }
}

Future<List<Post>> fetchPosts() async {
  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/posts'),
  );

  if (response.statusCode == 200) {
    final List data = jsonDecode(response.body);
    return data.map((e) => Post.fromJson(e)).toList();
  } else {
    throw Exception('请求失败');
  }
}
```

### 7.2 数据展示

```dart
FutureBuilder<List<Post>>(
  future: fetchPosts(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return const Center(child: CircularProgressIndicator());
    }

    if (snapshot.hasError) {
      return Center(child: Text('错误: ${snapshot.error}'));
    }

    final posts = snapshot.data ?? [];
    return ListView.builder(
      itemCount: posts.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(posts[index].title),
          subtitle: Text('ID: ${posts[index].id}'),
        );
      },
    );
  },
)
```

### 7.3 状态管理入门

初学者先理解这一点就够了：

- `setState` 适合页面内的小状态
- `Provider`、`Riverpod`、`Bloc` 适合更复杂的项目

最重要的思维是：

> 状态变了，界面就要跟着变。

```dart
setState(() {
  count++;
});
```

---

## 8. 从零开发一个 Flutter 项目

这里以“待办事项 App”为例，带你理解真实开发流程。

### 8.1 明确需求

一个简单的 Todo App 需要：

- 添加待办事项
- 删除待办事项
- 标记完成/未完成
- 页面刷新

### 8.2 设计数据模型

```dart
class TodoItem {
  final String title;
  bool done;

  TodoItem({required this.title, this.done = false});
}
```

### 8.3 先搭 UI，再加逻辑

真实项目建议这样做：

1. 先画页面
2. 再放静态数据
3. 再接交互
4. 再接网络或存储
5. 最后优化和打包

### 8.4 建议目录结构

```text
lib/
  main.dart
  models/
    todo_item.dart
  pages/
    home_page.dart
    add_todo_page.dart
  widgets/
    todo_tile.dart
  services/
    storage_service.dart
  providers/
    todo_provider.dart
```

### 8.5 完整 Demo

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.blue),
      home: const TodoPage(),
    );
  }
}

class TodoItem {
  String title;
  bool done;

  TodoItem({required this.title, this.done = false});
}

class TodoPage extends StatefulWidget {
  const TodoPage({super.key});

  @override
  State<TodoPage> createState() => _TodoPageState();
}

class _TodoPageState extends State<TodoPage> {
  final List<TodoItem> items = [
    TodoItem(title: '学习 Dart'),
    TodoItem(title: '学习 Flutter 基础'),
  ];

  final TextEditingController controller = TextEditingController();

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }

  void addItem() {
    final text = controller.text.trim();
    if (text.isEmpty) return;

    setState(() {
      items.add(TodoItem(title: text));
      controller.clear();
    });
  }

  void toggleItem(int index) {
    setState(() {
      items[index].done = !items[index].done;
    });
  }

  void deleteItem(int index) {
    setState(() {
      items.removeAt(index);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Todo List')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: controller,
                    decoration: const InputDecoration(
                      labelText: '输入待办事项',
                      border: OutlineInputBorder(),
                    ),
                  ),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: addItem,
                  child: const Text('添加'),
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: items.length,
              itemBuilder: (context, index) {
                final item = items[index];
                return ListTile(
                  leading: Checkbox(
                    value: item.done,
                    onChanged: (_) => toggleItem(index),
                  ),
                  title: Text(
                    item.title,
                    style: TextStyle(
                      decoration: item.done
                          ? TextDecoration.lineThrough
                          : TextDecoration.none,
                    ),
                  ),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete),
                    onPressed: () => deleteItem(index),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### 8.6 这个 Demo 的关键点

你要重点看懂这几件事：

- `TextField` 负责输入
- `TextEditingController` 读取输入内容
- `setState()` 负责刷新界面
- `ListView.builder()` 负责渲染列表
- `Checkbox` 负责完成状态
- `IconButton` 负责删除

---

## 9. 零基础 30 天学习计划

下面这个计划适合完全新手。建议每天学习 1 到 2 小时，重在动手。

### 第 1 周：Dart 入门

#### 第 1 天
- 认识 Flutter 和 Dart
- 安装环境
- 运行第一个 Hello World

#### 第 2 天
- 变量、类型、`var`、`final`、`const`
- 练习输出字符串和基本运算

#### 第 3 天
- 函数、参数、返回值
- 写加减乘除函数

#### 第 4 天
- 类、对象、构造函数
- 写一个 `User` 类

#### 第 5 天
- 列表、Map、Set
- 遍历集合

#### 第 6 天
- 空安全
- 可空类型和非空断言

#### 第 7 天
- 异步基础
- `Future`、`async/await`

---

### 第 2 周：Flutter 基础组件

#### 第 8 天
- Flutter 项目结构
- `main.dart`、`MaterialApp`、`Scaffold`

#### 第 9 天
- `Text`、`Icon`、`Image`
- 学会给文字和图片加样式

#### 第 10 天
- `Container`
- 边距、内边距、圆角、阴影

#### 第 11 天
- `Row`、`Column`
- 横向和纵向布局

#### 第 12 天
- `Stack`、`Positioned`
- 学会层叠布局

#### 第 13 天
- `ListView`、`GridView`
- 学会列表渲染

#### 第 14 天
- 做一个静态首页
- 仿照一个简单 App 页面

---

### 第 3 周：交互与页面开发

#### 第 15 天
- 按钮组件
- 点击事件

#### 第 16 天
- `TextField`
- 输入框和控制器

#### 第 17 天
- 表单 `Form`
- 校验逻辑

#### 第 18 天
- 页面跳转
- `Navigator.push` 和 `Navigator.pop`

#### 第 19 天
- 练习做两个页面之间的跳转

#### 第 20 天
- `setState` 和状态刷新
- 做一个计数器页面

#### 第 21 天
- 做一个简单 Todo 页面
- 学会增删改查的基础思路

---

### 第 4 周：项目能力提升

#### 第 22 天
- JSON 解析
- 把接口数据转成对象

#### 第 23 天
- 网络请求
- 拉取列表数据

#### 第 24 天
- `FutureBuilder`
- 展示加载状态和错误状态

#### 第 25 天
- 本地存储基础
- 了解 `shared_preferences` 或 `hive`

#### 第 26 天
- 状态管理入门
- 了解 `Provider`

#### 第 27 天
- 拆分组件
- 学会把页面拆小

#### 第 28 天
- 项目目录整理
- 设计合理的文件结构

#### 第 29 天
- 继续完善你的 Todo 或列表项目
- 加载空状态、删除确认、样式优化

#### 第 30 天
- 完整复盘
- 整理你的笔记和代码
- 准备进入下一个项目

---

## 10. 常见误区与学习建议

### 常见误区

1. **一上来就学复杂状态管理**
   - 建议先把 `setState` 学明白。

2. **只看教程不动手**
   - Flutter 非常依赖实践，必须自己写。

3. **不会拆组件**
   - 页面一复杂就要拆分组件。

4. **忽视 Dart 基础**
   - Dart 不熟，Flutter 会很痛苦。

5. **一开始就追求“完整 App”**
   - 先做小页面，再做小项目，再做完整项目。

### 学习建议

- 每天都写代码
- 每个知识点都做一个小 Demo
- 先模仿，再改造，再自己写
- 遇到问题时先看 Widget 树和状态流转

---

## 11. 总结

Flutter 的核心学习路线可以概括为：

**Dart 基础 → Widget 思维 → 布局与交互 → 状态管理 → 网络请求 → 本地存储 → 项目实战 → 性能优化**

你只要记住三句话：

1. Flutter 是用 Widget 组合 UI。
2. Dart 是 Flutter 的基础。
3. 学 Flutter 最好的方式是“边学边做”。

如果你愿意，下一步我可以继续带你做这三件事中的任意一件：

- 给你拆解一个**真正的新手 Flutter 项目目录**
- 带你从零写一个**完整 Flutter App**
- 再给你补一份**Flutter 面试/复习速查表**
