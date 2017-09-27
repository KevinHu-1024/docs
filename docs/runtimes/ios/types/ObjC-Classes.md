---
nav-title: "Objective-C Classes"
title: "Objective-C Classes"
description: "Describes how Objective-C classes are exposed."
position: 0
---

# Objective-C Classes
# Objective-C 中的类（Class）
Objective-C classes are exposed as JavaScript classes. Each Objective-C class is presented by a pair of corresponding JavaScript constructor function and a prototype object.

Objective-C 中的类被暴漏为 JavaScript 的类。每个 Objective-C 的类被展示位为一组相应的 JavaScript 构造函数和原型对象。

The methods declared in the Objective-C classes are exposed:
  * If static - on the JavaScript constructor function
  * If instance - on the JavaScript prototype object

定义在 Objective-C 类上的方法，按照下面两种规则进行暴漏：
  * 静态方法：暴漏在 JavaScript 构造函数上
  * 实例方法：暴漏在 JavaScript 原型对象上

For Objective-C properties, JavaScript property descriptors are declared on the prototype object.

对于 Objective-C 中的属性（properties）而言，它将通过 JavaScript 的属性描述符被定义在原型对象上。

## Prototype Chain
## 原型链
The prototype chain of the JavaScript objects matches the inheritance chain of the represented Objective-C classes. For example:

JavaScript 对象的原型链遵循 Objective-C 中类的继承链。例如：

```objective-c
@interface NSObject
+ (instancetype)alloc;
- (instancetype)init;
@end

@interface BaseClass : NSObject
+ (void)baseStaticMethod;
- (void)baseInstanceMethod;
@end

@interface DerivedClass : BaseClass
+ (void)derivedStaticMethod;
- (void)derivedInstanceMethod;
@end
```

Will generate the following JavaScript inheritance chain:

上面的例子将会生成下列 JavaScript 继承链：

```javascript
function NSObject() { /* native call */ };
// Object.getPrototypeOf(NSObject) === Function.prototype
NSObject.alloc = function () { /* native call */ };

// Object.getPrototypeOf(NSObject.prototype) === Object.prototype
NSObject.prototype.init = function () { /* native call */ };

function BaseClass() { /* native call */ };
Object.setPrototypeOf(BaseClass, NSObject);
BaseClass.baseStaticMethod = function () { /* native call */ };

BaseClass.prototype = Object.create(NSObject.prototype, { constructor: BaseClass });
BaseClass.prototype.baseInstanceMethod = function () { /* native call */ };

function DerivedClass() { /* native call */ };
Object.setPrototypeOf(DerivedClass, BaseClass);
DerivedClass.derivedStaticMethod = function () { /* native call */ };

DerivedClass.prototype = Object.create(NSObject.prototype, { constructor: DerivedClass });
DerivedClass.prototype.derivedInstanceMethod = function () { /* native call */ };
```

### JavaScript `instanceof` Opereator
### JavaScript 中的 `instanceof` 操作符

You can use the JavaScript [`instanceof` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) to see if an object inherits from a given class:

你可以使用 JavaScript 中的 [`instanceof` 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) 来查看一个对象是否继承于指定的类

```javascript
var object = DerivedClass.alloc().init();
console.log(object instanceof DerivedClass); // true
console.log(object instanceof BaseClass); // true
console.log(object instanceof NSObject); // true
console.log(object instanceof Object); // true
```

## Instantiating Objects
## 实例对象

### `alloc`, `init` or `new`
### `alloc`, `init` 和 `new`
Objective-C instances are created using:

Objective-C 实例用以下方式创建：

```objective-c
UIView *view1 = [[UIView alloc] init];
// 你也可以使用简写
UIView *view2 = [UIView new];
```

Which is exposed as:
这将被暴漏为：

```javascript
var view1 = UIView.alloc().init();
// 你也可以使用简写
var view2 = UIView.new();
```

### JavaScript `new` Operator
### JavaScript `new` 操作符
The [`new` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) used with JavaScript constructor function for Objective-C class will try to match an appropriate initializer based on the number and types of the arguments. However when a class has more complex initializers, it is impossible to unambiguously select one at runtime. So we would like to discourage you from using `new`, but still in some simple cases it is more convenient:

[`new` 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) 在 JavaScript 中与构造函数一起使用，在 Objective-C 中则会根据参数的数量和类型去匹配一个 initializer。但是，当一个类有很多复杂的 initializer 时，就不可能明确地在运行时中选出来。所以我们不鼓励使用 `new`，但是在一些简单的使用场景下，使用 `new` 还是比较方便的。

```javascript
// Will call UIView.alloc().init();
var view1 = new UIView();

// Will call UIView.alloc().initWithFrame(...)
var view2 = new UIView(CGRectMake(10, 10, 200, 00));
```

```javascript
// 将会调用 UIView.alloc().init();
var view1 = new UIView();

// 将会调用 UIView.alloc().initWithFrame(...)
var view2 = new UIView(CGRectMake(10, 10, 200, 00));
```
## Methods
## 方法

When Objective-C methods are exposed in JavaScript, we remove the colons from their names (selector), and then upper-case the letters following the removed colons.

当 Objective-C 的方法在 JavaScript 中暴漏时，我们从他们的名字（选择器）上移除了 colons，并且使用大写字符来表示移除的 colons

For example:

例如：

```objective-c
@interface UIAlertView : UIView
- (void)dismissWithClickedButtonIndex:(NSInteger)buttonIndex animated:(BOOL)animated;
@end
```
Will form the following JavaScript instance method:
```javascript
var instance = UIAlertView.alloc().init();
instance.dismissWithClickedButtonIndexAnimated(0, true);
```

### Static Methods
### 静态方法

Static methods are exposed in JavaScript as functions that call the underlying native methods. They are defined as properties on the constructor function. Since the constructor functions of derived classes have for prototype their base class constructor function, static methods are inherited.

静态方法在 JavaScript 中以函数的形式暴漏出来，它会调用底层原生的方法。它们被定义在构造函数的属性上面。当构造函数的父类含有静态方法时，这些静态方法也将被这个构造函数继承。

For example:

例如：

```objective-c
@interface BaseClass : NSObject
+ (void)baseStaticMethod;
@end

@interface DerivedClass : BaseClass
+ (void)derivedStaticMethod;
@end
```

You can call them in JavaScript:

在 JavaScript 中，你可以这样调用它们：

```javascript
BaseClass.baseStaticMethod();

DerivedClass.baseStaticMethod();
DerivedClass.derivedStaticMethod();
```

If `DerivedClass` overrides `baseStaticMethod` the correct override will be called, even if it is not declared in the header.

如果 `DerivedClass` 重写了 `baseStaticMethod`，那么当调用时，覆盖的方法将被正确地调用。即使这个函数在头文件中没有定义，也可以被正确调用。

### Instance Methods
### 实例方法

Instance methods are exposed in JavaScript as functions that call the underlying native methods. They are defined as properties on the prototype object. Since the prototype objects of derived classes have for prototype their base class' prototype, instance methods are inherited.

实例方法在 JavaScript 中以函数暴漏，它会调用底层的原生方法。它们作为对象的属性被定义在原型对象中。当这个原型对象的父类含有实例方法时，这些实例方法也将被这个原型对象继承。

For example:

例如：

```objective-c
@interface BaseClass : NSObject
- (void)baseInstanceMethod;
@end

@interface DerivedClass : BaseClass
- (void)derivedInstanceMethod;
@end
```

You can call them in JavaScript:

在 JavaScript 中，你可以这样调用它们：

```javascript
var baseInstance = BaseClass.alloc().init();
baseInstance.baseInstanceMethod();

var derivedInstance = DerivedClass.alloc().init();
derivedInstance.baseInstanceMethod();
derivedInstance.derivedInstanceMethod();
```

If `DerivedClass` overrides `baseInstanceMethod` the correct override will be called, even if it is not declared in the header.

如果 `DerivedClass` 重写了 `baseInstanceMethod`，那么当调用时，覆盖的方法将被正确地调用。即使这个函数在头文件中没有定义，也可以被正确调用。

## Properties
## 属性

Objective-C properties are exposed as JavaScript property descriptors. For example consider the JavaScript objects generated for the following Objective-C interface:

Objective-C 中的属性被暴漏为 JavaScript 的属性描述符。举一个例子，假设有一个 JavaScript 对象，是按照下面这段 Objective-C 接口文件创建出来的：

```objective-c
@interface UIAlertView
@property (class) NSString *layerClass;
@property (nonatomic, copy) NSString *title;
@end
```

The `UIAlertView` constructor function will have a property `layerClass` that when get or assigned will call the native Objective-C getter and setter methods. Similarly the `UIAlertView` prototype will have a `title` property:

`UIAlertView` 构造函数将会拥有一个 `layerClass` 属性，当对其进行取值或赋值时，将会调用原生 Objective-C 的 getter 和 setter 方法。类似地，`UIAlertView` 原型对象也会拥有一个 `title` 属性。

```javascript
Object.defineProperty(UIAlertView, "layerClass", {
    get: function () { /* native call */ },
    set: function (newLayerClass) { /* native call  */ }
});

Object.defineProperty(UIAlertView.prototype, "title", {
    get: function () { /* native call */ },
    set: function (newTitle) { /* native call  */ }
});
```

You can use it in JavaScript:
在 JavaScript 中可以这样使用：

```javascript
console.log(UIAlertView.layerClass); // The layer class of UIAlertView

var instance = UIAlertView.alloc.init();
instance.title = "The title";
console.log(instance.title); // "The title"
```

In Objective-C the getter methods by default have the name of the property and the setter methods have for name, the property name prefixed with "set". Because of the collisions the getter and setter methods for properties are not exposed as JavaScript functions.

在 Objective-C 中，getter 方法默认为属性的名字，而 setter 方法为属性名加“set”前缀。因为 getter 和 setter 的函数名冲突，在 JavaScript 中无法进行暴漏。

```objective-c
@interface UIAlertView
- (NSString *)title;
- (void)setTitle:(NSString *)newTitle;·
@end
```

In Objective-C you can specify custom getter/setter method names. In this case the specified getter/setter methods will be called by the property descriptor and are not exposed in JavaScript.

在 Objective-C 中，你可以自定义 getter/setter 方法的名字。因此被声明的 getter/setter 函数将通过属性描述符调用，不会在 JavaScript 中暴漏。

## Inheriting Native Classes in JavaScript
## 在 JavaScript 中继承原生类

[You can subclass Objective-C classes in JavaScript.](../how-to/ObjC-Subclassing.md)

[你可以在 JavaScript 中子类化一个 Objective-C 的类 ](../how-to/ObjC-Subclassing.md)
