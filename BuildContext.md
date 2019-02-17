# BuildContext

​	对于Flutter来说，Everything is Widget，而我们使用的最多的就是StatelessWidget喝StatefulWidget，StatelessWidget需要通build(BuildContext context)构建展示的UI，StatefulWidget需要State的实现类中的build(BuildContext context)来构建UI，那么这个BuildContext是否和Android中的Context是一类东西呢？

​	先来看看源码：

```dart
/// The [BuildContext] for a particular widget can change location over time as
/// the widget is moved around the tree. Because of this, values returned from
/// the methods on this class should not be cached beyond the execution of a
/// single synchronous function.
///
/// [BuildContext] objects are actually [Element] objects. The [BuildContext]
/// interface is used to discourage direct manipulation of [Element] objects.
abstract class BuildContext {
  /// The current configuration of the [Element] that is this [BuildContext].
  Widget get widget;

  /// The [BuildOwner] for this context. The [BuildOwner] is in charge of
  /// managing the rendering pipeline for this context.
  BuildOwner get owner;
```

这是一个抽象类，并没有具体的方法实现。

​	下面以StatelessWidget为例看看Flutter是如何构建UI的，而BuildContext在其中起了什么作用，

```dart
abstract class StatelessWidget extends Widget {
  /// Initializes [key] for subclasses.
  const StatelessWidget({ Key key }) : super(key: key);

  /// Creates a [StatelessElement] to manage this widget's location in the tree.
  ///
  /// It is uncommon for subclasses to override this method.
  @override
  StatelessElement createElement() => StatelessElement(this);
    
  @protected
  Widget build(BuildContext context);
}
```

StatelessWidget中的代码很少，其中build方法就是交给开发者实现的，它自己只是在createElement方法中创建了一个StatelessElement，并将StatelessWidget自身传了过去。

```dart
// An [Element] that uses a [StatelessWidget] as its configuration.
class StatelessElement extends ComponentElement {
  /// Creates an element that uses the given widget as its configuration.
  StatelessElement(StatelessWidget widget) : super(widget);

  @override
  StatelessWidget get widget => super.widget;

  @override
  Widget build() => widget.build(this);

  @override
  void update(StatelessWidget newWidget) {
    super.update(newWidget);
    assert(widget == newWidget);
    _dirty = true;
    rebuild();
  }
}
```

StatelessElement也有一个build方法，它的实现只是调用了StatelessWidget的build方法，并传入自身，但是StatelessWidget的build中是BuildContext参数，而此时传入的则是一个StatelessElement参数，接下来看它的父类：

```dart
abstract class ComponentElement extends Element
abstract class Element extends DiagnosticableTree implements BuildContext
```

确实最终实现了BuildContext抽象类。

​	而在官方对于BuildContext的说明：

**BuildContext**objects are actually **Element** objects. The **BuildContext**interface is used to discourage direct manipulation of **Element** objects.

BuildContext对象实际上就是Element对象，BuildContext 接口用于阻止对 Element 对象的直接操作。

​	因此虽然我们说Everything is Widget，但实际上真正的UI应该是Element，只是我们操作的是Widget，而Widget到Element之间的转化则是由Flutter底层进行的处理，Widget和Element之间的关联则就是通过BuildContext来做到的。