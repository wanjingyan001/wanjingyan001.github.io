# flutter导航学习记录

### Navigator的基础使用

**push**

```dart
Navigator.of(context).push(Route<T> route);
//等同于
Navigator.push(BuildContext context,Route<T> route);
```

这是flutter导航的最基本用法,大多数时候效果等同于Android中的

```kotlin
val intent = Intent(context, MainActivity::class.java)
startActivity(intent)
```

效果都是将一个新页面放入路由(页面)栈中![Stack after pushing page1](/Users/admin/dev/mybook/QQ20190123-152727.png)

route参数通常需要传入一个MaterialPageRoute或者CupertinoPageRoute的实例，它们都需要传递一个

```dart
typedef WidgetBuilder = Widget Function(BuildContext context)
```

WidgetBuilder返回的widget就是入栈的新页面。

**pop**

```dart
Navigator.of(context).canPop();
Navigator.of(context).maybePop([ T result ]);
Navigator.of(context).pop([ T result ]); 
```

canPop()用于判断当前路由能否pop出栈，maybePop()会根据canPop进行不同处理，canPop为true时，当前路由将pop出栈，false时将不进行动作。pop()会将当前路由强制出栈，若此时栈底没有路由则退出程序。[T result]是路由出栈时返回给上一个路由的值，如果接收这个值将在另一篇记录。

### 根据路由名字导航

```dart
MaterialApp(
//      initialRoute: '/screen1',
      home: new Screen1(),
      routes: <String, WidgetBuilder>{
        '/screen1': (BuildContext context) => new Screen1(),
        '/screen2': (BuildContext context) => new Screen2(),
        '/screen3': (BuildContext context) => new Screen3(),
        '/screen4': (BuildContext context) => new Screen4(),
        '/screen5': (BuildContext context) => new Screen5('userName null'),
      },
    )
```

​	要使用路由名字进行导航，需要使用flutter提供的MaterialApp，其中routes参数要传递的是一个map，这个map就是提前定义好的路由表，所有需要使用路由名字进行导航的页面，都需要提前在路由表中注册好。initialRoute和home起到的是同样的作用，即设置栈底路由，不同的是initialRoute使用路由表中注册好的路由名字，而home则直接使用widget实例。

- **pushNamed(String routeName)**

  ```dart
  Navigator.of(context).pushNamed('/screen2');
  //等同于
  Navigator.of(context).push(MaterialPageRoute(builder: (_) {
                    return Screen2();
                  }));
  ```

  此时的路由栈![stack](/Users/admin/dev/mybook/QQ20190123-171044.png)  ，继续向栈中push页面screen3![stack](/Users/admin/dev/mybook/QQ20190123-171238.png)

- **pushReplcaementNamed，popAndPushNamed**

  此时在screen3页面调用

  ```dart
  Navigator.of(context).pushReplacementNamed('/screen4',result: T)；
  //或者
  Navigator.of(context).popAndPushNamed('/screen4',result: T);
  ```

  此时的路由栈是![stack](/Users/admin/dev/mybook/QQ20190123-174817.png),若是传入了result参数，此时screent2页面可以在路由导航方法后通过.then((result){})获取到result。目前这两个方法之间只发现了页面切换动画的区别，pushReplacement的动画是直接启动一个符合当前系统的页面入栈动画，popAndPushNamed的动画则是先页面pop出栈在push入栈一个新页面的动画。

### 多路由出栈

- **pushAndRemoveUntil**

  ​	当我们在程序中打开多个页面后需要进行注销操作时，在重新登录的页面此时点击返回应该是直接退出应用，在Android中可以通过launchMode或者intent.flags进行类似的设置，即push入栈一个新路由后删除之前的所有路由。

  ```dart
  Navigator.of(context).pushNamedAndRemoveUntil('/screen4',
  (Route<dynamic> route) => false);
                  //and
  Navigator.of(context).pushAndRemoveUntil(
  MaterialPageRoute(builder: (context) => Screen4()), (_) => false);
  ```

   将screen1,screen2,screen3依次push入栈之后，调用上面的导航代码，则路由栈会进行以下图示的变化：![路由栈变化](/Users/admin/dev/mybook/QQ20190125-105625.png)此时在screen4页面点击返回键将推出程序。

  ​	如果此时我们不希望删除所有screen4以下的路由，而是希望删除screen至screen4之间的所有路由，则需要修改

  ```dart
  Future<T> pushAndRemoveUntil<T extends Object>(Route<T> newRoute, RoutePredicate predicate)
  ```

  中的Predicate参数，这是一个返回值为bool类型的函数

  ```dart
  typedef RoutePredicate = bool Function(Route<dynamic> route)
  ```

  将原来的(Route<dynamic> route) => false修改为ModalRoute.withName('Screen1')。

- **popUntil**

  ​	pushAndRemoveUntil会先push入栈一个路由，如果不想push入栈，则可以直接使用

  popUntil。