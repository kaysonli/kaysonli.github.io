---
title: JavaScript Class 完全指南
date: 2020-01-03 13:04:56
tags: JavaScript
---


JavaScript 使用原型继承：每个对象都从原型对象继承属性和方法。

Java 或 Swift 等语言中作为*创建对象的蓝图*的传统 Class，在JavaScript 中不存在。原型继承只处理对象。

原型继承可以模拟经典的类继承。为了将传统的类引入 JavaScript，ES2015 标准引入了`class`语法：这是在原型继承之上的一种语法糖。

这篇文章让你熟悉 JavaScript 类：如何定义类，初始化实例，定义字段和方法，理解私有和公共字段，掌握静态字段和方法。
<!-- more -->

## 1\. 定义： *class* 关键字

JavaScript 关键字`class` 用于定义类：
```
class User {
  // The body of class
}
```

上面的代码定义了一个类 `User`。花括号`{ }`标记 class 主体。注意，这个语法叫做 *class 声明*。

可以不指定类名，你可以通过使用 *class 表达式* 把 class 分配给一个变量：
```
const UserClass = class {
  // The body of class
};
```

您可以轻松地将 class 导出为 ES2015 模块的一部分。下面是一个“默认导出”的语法：
```
export default class User {
 // The body of class
}
```

具名导出：
```
export class User {
  // The body of class
}
```

当你创建一个*实例*时，class 就变得非常有用。实例就是包含了 class 所描述的数据和行为的一个对象。
![](https://dmitripavlutin.com/static/262db61c59f663901d75fbe5c60ec28d/fa9e1/instances-3.png) 

 JavaScript 的`new` 操作符用于实例化 class ：`instance = new Class()`。

例如，你可以用`new` 操作符实例化`User` 类：
```
const myUser = new User();
```

`new User()` 创建了 `User` 类的一个实例。

## 2\. 构造函数： *constructor()*

`constructor(param1, param2, ...)` 是在 class 内部初始化实例的一个特殊方法。这是设置字段初始值或进行对象设置的地方。

下面的例子就是在构造函数里设置`name`字段的初始值：
```
class User {
  constructor(name) {    this.name = name;  }}
```

`User`的构造函数有一个参数`name`，用于设置字段`this.name`的初始值。

构造函数内部的 `this` 值等于新创建的实例。

用来实例化类的参数变成了构造函数的参数：
```
class User {
  constructor(name) {
    name; // => 'Jon Snow'    this.name = name;
  }
}

const user = new User('Jon Snow');
```

构造函数内部的`name` 参数的值是`'Jon Snow'`。

如果不定义类的构造函数，就会创建默认构造函数。默认构造函数是一个空函数，不会修改实例。

同时，JavaScript 类最多只能有一个构造函数。
## 3\. 字段

类字段是保存信息的变量。字段可以附属于两种实体：
1.  class 实例字段
2.  class 自有字段（即静态字段）

字段有两种级别的可访问性：
1.  公有：字段可任意访问
2.  私有：只能在 class 内部访问

### 3.1 公有实例字段

让我们看看之前的代码：
```
class User {
  constructor(name) {
    this.name = name;  }
}
```

表达式`this.name = name`创建了一个实例字段`name`并设置了初始值。

之后就可以通过属性的形式访问 `name` 字段：
```
const user = new User('Jon Snow');
user.name; // => 'Jon Snow'
```

`name`是一个*公有字段*，因为你可以在`User` 类外部访问到它。

当字段在构造函数中隐式创建时，就像前面的例子一样，可能很难管理字段列表。你必须从构造函数的代码中破译它们。

更好的方式是显式地声明 class 字段。无论构造函数做什么，实例总是具有相同的字段列表。

[class 字段提案](https://github.com/tc39/proposal-class-fields) 允许你在 class 主体中定义字段。另外，你可以立即指定初始值：
```
class SomeClass {
  field1;  
  field2 = 'Initial value';
  // ...
}
```

让我们修改 `User` 类，声明一个公有字段`name`：
```
class User {
  name;  
  constructor(name) {
    this.name = name;
  }
}

const user = new User('Jon Snow');
user.name; // => 'Jon Snow'
```
 class 主体里的`name;`声明了一个公有字段`name`。

以这种方式声明的公共字段很有表现力：快速查看字段声明就足以知晓类的数据结构。

而且，类字段可以在声明时立即初始化。
```
class User {
  name = 'Unknown';
  constructor() {
    // No initialization
  }
}

const user = new User();
user.name; // => 'Unknown'
```

class 主体内的 `name = 'Unknown'` 声明了一个 `name` 字段并设置了初始值`'Unknown'`。

对公有字段的访问和更新没有限制。可以在构造函数、方法以及 class 外部读取和赋值给公有字段。

### 3.2 私有实例字段

封装是一个重要的概念，它可以隐藏 class 内部的细节。使用封装类的人只依赖类提供的公共接口，而不与类的实现细节耦合。

组织 class 的时候充分考虑封装，当实现细节改变的时候更新起来更容易。

隐藏对象内部数据的一种好方法是使用私有字段。这些字段只能在它们所属的类中读取和更改。类的外部不能直接更改私有字段。
> *私有字段* 只能在 class 内部访问。

在字段名前面加上特殊字符 `#` 可以使其变为私有，比如`#myField`。每次使用该字段时，前缀`#` 必须保留：声明时、读取时和修改时。

让我们确保字段 `#name`可以在实例初始化时设置一次：
```
class User {
  #name;
  constructor(name) {
    this.#name = name;
  }

  getName() {
    return this.#name;
  }
}

const user = new User('Jon Snow');
user.getName(); // => 'Jon Snow'

user.#name;     // 抛出 SyntaxError 异常
```

`#name` 是私有字段。你可以在 `User`内部访问和修改 `#name`。 `getName()` 方法可以访问私有字段 `#name`。

但是如果你尝试从 `User` 类外部访问私有变量`#name`，就会抛出语法错误：`SyntaxError: Private field '#name' must be declared in an enclosing class`。
### 3.3 公有静态字段

你也可以在 class 自己上面定义字段：*静态字段*。这有助于定义类常量或存储特定于该类的信息。

要在 JavaScript 类中创建静态字段，请使用特殊的关键字`static`加上字段名：`static myStaticField`。

让我们添加一个新的字段`type`，表示用户类型：admin 或 regular。静态字段 `TYPE_ADMIN`和 `TYPE_REGULAR`是区分用户类型的常量：
```
class User {
  static TYPE_ADMIN = 'admin';  static TYPE_REGULAR = 'regular';
  name;
  type;

  constructor(name, type) {
    this.name = name;
    this.type = type;
  }
}

const admin = new User('Site Admin', User.TYPE_ADMIN);
admin.type === User.TYPE_ADMIN; // => true
```


`static TYPE_ADMIN` 和`static TYPE_REGULAR` 在`User` 类内部定义了静态变量。要访问静态字段，你必须用类名加上字段名：`User.TYPE_ADMIN` 和`User.TYPE_REGULAR`
### 3.4 私有静态字段

有时甚至静态字段也是你希望隐藏的实现细节。在这里，你也可以将静态字段设为私有。

要将静态字段设为私有，只要在字段名前面加上特殊符号`#` ：`static #myPrivateStaticField`。

假设你想限制 `User`类的实例数量。为了隐藏实例限制的细节，你可以创建私有静态字段：
```
class User {
  static #MAX_INSTANCES = 2;  static #instances = 0;  
  name;

  constructor(name) {
    User.#instances++;
    if (User.#instances > User.#MAX_INSTANCES) {
      throw new Error('Unable to create User instance');
    }
    this.name = name;
  }
}

new User('Jon Snow');
new User('Arya Stark');
new User('Sansa Stark'); // throws Error
```

静态字段 `User.#MAX_INSTANCES`设置了允许的最大实例数量，静态字段`User.#instances`是实际创建的实例数量。

私有静态字段只能在 `User` 类内部访问。外部范围无法干预这里的限制机制：这就是封装的好处。

## 4\. 方法

字段包含了数据。但是修改数据的能力是由特殊函数提供的，它是类的一部分：*方法*。

JavaScript 类支持实例方法和静态方法。
### 4.1 实例方法

实例方法可以访问和修改实例数据。实例方法可以调用其他实例方法，也可以调用任意静态方法。

例如，我们在 `User` 类中定义一个 `getName()` 方法，用来返回 `name`：
```
class User {
  name = 'Unknown';

  constructor(name) {
    this.name = name;
  }

  getName() {    return this.name;  }}

const user = new User('Jon Snow');
user.getName(); // => 'Jon Snow'
```

`getName() { ... }` 是`User` 类中的一个方法。`user.getName()` 是一个方法调用：它会执行该方法并返回计算后的值，如果有的话。

在类的方法和构造函数中，`this` 的值等于类的实例。可用`this`访问实例数据：`this.field`，或者调用其他方法：`this.method()`。

我们来添加一个`nameContains(str)`方法，它接受一个参数，并调用另一个方法：
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }

  nameContains(str) {   
 return this.getName().includes(str);  }}

const user = new User('Jon Snow');
user.nameContains('Jon');   // => true
user.nameContains('Stark'); // => false
```

`nameContains(str) { ... }`是 `User`类的一个方法，接受一个参数`str`。另外，它还执行了实例的另一个方法`this.getName()`，来获取用户的名字。

方法也可以是私有的。要将方法变为私有，在名字前加上 `#`前缀即可：

```
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }

  #getName() {    return this.#name;  }
  nameContains(str) {
    return this.#getName().includes(str);  }
}

const user = new User('Jon Snow');
user.nameContains('Jon');   // => true
user.nameContains('Stark'); // => false

user.#getName(); // SyntaxError is thrown
```

`#getName()`是个私有方法。在方法`nameContains(str)`内部，用这种方式调用私有方法：`this.#getName()`。

由于是私有的，`#getName()`不能在`User` 类外部被调用。
### 4.2 getters 和  setters

getter 和 setter 模拟常规字段，但对如何访问和更改字段有更多的控制。

getter 在试图获取字段值时执行，而 setter 在试图设置值时执行。

为了确保 `User` 的 `name`属性不为空，让我们在 getter 和 setter 中包装私有字段`#nameValue` ：
```
class User {
  #nameValue;

  constructor(name) {
    this.name = name;
  }

  get name() {   
     return this.#nameValue;
  }

  set name(name) {    
    if (name === '') {
      throw new Error(`name field of User cannot be empty`);
    }
    this.#nameValue = name;
  }
}

const user = new User('Jon Snow');
user.name; // The getter is invoked, => 'Jon Snow'
user.name = 'Jon White'; // The setter is invoked

user.name = ''; // The setter throws an Error
```

`get name() {...}` getter 在你访问字段 `user.name`时执行。

而 `set name(name) {...}`  在字段更新`user.name = 'Jon White'`时执行。如果新的值是空字符串，setter 就会抛出错误。
### 4.3 静态方法

静态方法是直接附属于类的方法。它们包含了跟类相关的逻辑，而不是类的实例。

要创建静态方法，请使用特殊的关键字`static` ，后面加上常规的方法语法：`static myStaticMethod() { ... }`

使用静态方法时，需要记住两个简单的规则：
1.  静态方法可以访问静态字段
2.  静态方法不能访问实例字段

例如，我们来创建一个静态方法，用于检测某个用户名是否被占用。
```
class User {
  static #takenNames = [];

  static isNameTaken(name) {   
    return User.#takenNames.includes(name);  
  }
   name = 'Unknown';

   constructor(name) {
    this.name = name;
    User.#takenNames.push(name);
  }
}

const user = new User('Jon Snow');

User.isNameTaken('Jon Snow');   // => true
User.isNameTaken('Arya Stark'); // => false
```

`isNameTaken()`是个静态方法，使用了静态私有字段`User.#takenNames` 检查被占用的名字。

静态方法可以是私有的：`static #staticFunction() {...}`。同样，它们也遵循私有规则：只能在类内部调用私有静态方法。
## 5\. 继承： *extends*

JavaScript 类使用 `extends` 关键字支持单继承。

语句 `class Child extends Parent { }` 中， `Child`类继承`Parent` 类的构造函数、字段和方法。

例如，让我们创建一个子类 `ContentWriter`，继承自父类 `User`。
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {  posts = [];
}

const writer = new ContentWriter('John Smith');

writer.name;      // => 'John Smith'
writer.getName(); // => 'John Smith'
writer.posts;     // => []
```

`ContentWriter`从 `User` 继承了构造函数、方法`getName()`和字段`name`。同时，`ContentWriter`类还声明了一个新字段`posts`。

注意，父类的私有成员不能被子类继承。
### 5.1 父类构造函数：*constructor()* 中的 *super()*

如果你想在子类中调用父类的构造函数，你需要在子类构造函数中使用特殊的`super()`方法。

例如，我们让 `ContentWriter`的构造函数调用父类`User`的构造函数，同时初始化`posts `字段：
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {
  posts = [];

  constructor(name, posts) {
    super(name);    
    this.posts = posts;
  }
}

const writer = new ContentWriter('John Smith', ['Why I like JS']);
writer.name; // => 'John Smith'
writer.posts // => ['Why I like JS']
```

子类`ContentWriter` 中的`super(name)`执行了父类`User`的构造函数。

注意，在子类构造函数中必须在使用`this` 关键字之前调用`super()`。调用`super()`后才保证父类构造函数完成了实例化。 
```
class Child extends Parent {
  constructor(value1, value2) {
    // 这样是不行的
    this.prop2 = value2;    
     super(value1);  }
}
```

### 5.2 父类实例：方法中的 *super* 

如果你想在子类方法中访问父类方法，你可以使用特殊的快捷方式`super`。
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {
  posts = [];

  constructor(name, posts) {
    super(name);
    this.posts = posts;
  }

  getName() {
    const name = super.getName();    
    if (name === '') {
      return 'Unknwon';
    }
    return name;
  }
}

const writer = new ContentWriter('', ['Why I like JS']);
writer.getName(); // => 'Unknwon'
```

子类 `ContentWriter` 中的`getName()`访问了父类 `User`的方法 `super.getName()`。

该特性叫做[方法重写](https://en.wikipedia.org/wiki/Method_overriding).

注意，你也可以在静态方法中使用 `super` ，用于访问父类的静态方法。
## 6\. 对象类型检测：*instanceof*

`object instanceof Class` 是用来判断`object` 是否为 `Class`实例的操作符。

我们来看看`instanceof`实际是怎么用的：
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

const user = new User('Jon Snow');
const obj = {};

user instanceof User; // => true
obj instanceof User; // => false
```

`user`是 `User`类的一个实例，因此`user instanceof User`的值为`true`。

空对象`{}`不是 `User`的实例，相应的`obj instanceof User`就是`false`。

`instanceof` 是多态的：该操作符认为子类实例也是父类的实例。
```
class User {
  name;

  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class ContentWriter extends User {
  posts = [];

  constructor(name, posts) {
    super(name);
    this.posts = posts;
  }
}

const writer = new ContentWriter('John Smith', ['Why I like JS']);

writer instanceof ContentWriter; // => true
writer instanceof User;          // => true
```

`writer`是子类 `ContentWriter`的实例。操作符 `writer instanceof ContentWriter`是值为 `true`。

同时，`ContentWriter`是`User`的子类，因此 `writer instanceof User` 也是`true`。

如果要判断实例的确切类要怎么做？你可以使用 `constructor`属性，并与 class 直接比较：
```
writer.constructor === ContentWriter; // => true
writer.constructor === User;          // => false
```

## 7\. 类与原型

必须这样说，JavaScript 的 class 语法很好地抽象了原型继承机制。为了描述 `class`语法，我甚至没用到“prototype”这个词。

但是 class 是在原型继承的基础上构建的。每个类都是一个函数，并在作为构造函数调用时创建一个实例。

下面这两段代码是等效的。

class 版本：

```
class User {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

const user = new User('John');

user.getName();       // => 'John Snow'
user instanceof User; // => true
```

prototype 版本：

```
function User(name) {
  this.name = name;
}

User.prototype.getName = function() {
  return this.name;
}

const user = new User('John');

user.getName();       // => 'John Snow'
user instanceof User; // => true
```

如果您熟悉 Java 或 Swift 语言的经典继承机制，那么 class 语法更容易使用。

不管怎么样，即使你在 JavaScript 中使用 class 语法，我还是推荐你好好掌握[原型继承](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

## 8\. class 特性可用性

本文提到的 class 特性出现在 ES2015 和 stage 3 提案。

到 2019 年底，class 特性分布在以下几个提案和标准中：
*   公有和私有实例字段属于[Class 字段提案](https://github.com/tc39/proposal-class-fields)
*   私有实例方法和访问器属于[Class 私有方法提案](https://github.com/tc39/proposal-private-methods)
*   公有和私有静态字段和私有静态方法属于[Class 静态特性提案](https://github.com/tc39/proposal-static-class-features/)
*   其余的属于 ES2015 标准。

## 9\. 总结

JavaScript 类用构造函数初始化实例、定义字段和方法。你甚至可以使用`static`关键字在类上面附加字段和方法

继承是通过 `extends`关键字实现的：你可以轻松地从父类创建子类。`super` 关键字用于子类访问父类。

为了利用封装，让字段和方法变成私有以便隐藏 class 的内部细节。私有字段和方法名必须以`#`开头。

JavaScript 中的类变得越来越方便使用了。

*在私有属性前加上`#` 前缀，你怎么看？*
> 作者：Dmitri Pavlutin
原文：[https://dmitripavlutin.com/javascript-classes-complete-guide/](https://dmitripavlutin.com/javascript-classes-complete-guide/)
翻译：1024译站

