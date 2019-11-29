---
title: 深入浅出 JavaScript 原型链
date: 2019-11-26 09:49:42
tags: JavaScript
---

![](/uploads/1618526-3e4a041990941c42.webp)

> 了解原型链继承的概念

这篇文章，我们来学习一下 JavaScript 原型链。我们将了解对象之间是怎么关联的，以及如何实现对象之间的继承关系。

## 目标

作为开发人员，我们写代码的主要任务就是操作数据。我们获取数据并将其存储在某些位置，然后在数据上执行一些功能。

如果能将功能和相关数据绑在一块，那岂不是更好？这样对我们来说操作起来更容易。
<!-- more -->
假设有一个 `Player` 对象：

```
{
  userName: 'sag1v',
  score: '700'
}

```

如果想要在这个对象上执行一些功能，比如修改分数，该怎么做呢？我们要把`setScore`方法放在哪呢？
## 对象

当我们想保存相关数据的时候，经常会用到对象。对象就像一个箱子，可以往里面放各种相关的东西。

在深入了解之前，我们先要搞清楚 `Object` 到底是什么，以及创建对象的一些方式。

### 对象字面量

```
const player1 = {
  userName: 'sag1v',
  score: '700',
  setScore(newScore){
    player1.score = newScore;
  }
}

```

对象字面量表示法（或者叫“对象初始化器”）是一个表达式，每当表达式所在的语句执行时都会创建一个新对象。

我们也可以用点号和方括号来创建和访问对象的属性：
```
const player1 = {
  name: 'Sagiv',
}

player1.userName = 'sag1v';
player1['score'] = 700;
player1.setScore = function(newScore) {
  player1.score = newScore;
}

```

### Object.create

另外一种创建 `Object`的方式是使用 `Object.create` 方法：

```
const player1 = Object.create(null)
player1.userName = 'sag1v';
player1['score'] = 700;
player1.setScore = function(newScore) {
  player1.score = newScore;
}

```

`Object.create` 总是返回一个新的空对象，但如果我们传给它另一个对象，就会得到额外的功能。我们稍后会再讲。
## 自动化

显然，我们不想每次都手动创建这些对象，我们可能想让这个操作自动化。 因此，让我们写一个函数来为我们创建`Player`对象。
### 工厂方法

```
function createPlayer(userName, score) {
  const newPlayer = {
    userName,
    score,
    setScore(newScore) {
      newPlayer.score = newScore;
    }
  }
  return newPlayer;
}

const player1 = createPlayer('sag1v', 700);

```

这种模式通常称为“工厂方法”，有点像工厂中的传送带输出货物，我们传入相关参数并返回我们需要的`Object` 。

如果我们运行这个函数两次会发生什么？
```
function createPlayer(userName, score) {
  const newPlayer = {
    userName,
    score,
    setScore(newScore) {
      newPlayer.score = newScore;
    }
  }
  return newPlayer;
}

const player1 = createPlayer('sag1v', 700);
const player2 = createPlayer('sarah', 900);

```

我们会得到两个这样的对象：
```
{
  userName: 'sag1v',
  score: 700,
  setScore: ƒ
}

{
  userName: 'sarah',
  score: 900,
  setScore: ƒ
}

```

你注意到重复的地方了吗？

每个实例都存了一份`setScore` ，这就违反了D.R.Y原则(Don't Repeat Yourself，不要重复自己) 。

能不能在其他某个地方只存一份，但仍然能够通过对象实例访问：`player1.setScore(1000)`？

### OLOO - 链式对象（Objects Linked To Other Objects）

让我们回到 `Object.create`，我们说过它**总是**生成一个**空**对象，但如果给它传一个对象，我们会得到额外的功能。
```
const playerFunctions = {
  setScore(newScore) {
    this.score = newScore;
  }
}

function createPlayer(userName, score) {
  const newPlayer = Object.create(playerFunctions);
  newPlayer.userName = userName;
  newPlayer.score = score;
  return newPlayer;
}

const player1 = createPlayer('sag1v', 700);
const player2 = createPlayer('sarah', 900);

```

这段代码跟前面的代码很像，只有一个重要的差别，就是新的实例不再直接定义`setScore` ，而是**链接**到`playerFunctions`里的方法。

原来，JavaScript 里的**所有**对象都有一个特殊的隐藏属性 `__proto__` ，如果这个属性指向某个对象，JS 引擎就认为原对象也有这个对象上的属性。换句话说，每个对象可以通过`__proto__`属性链接到其他对象，并能访问上面的属性，就好像是自有的一样。

>**注意**：不要混淆了`__proto__`和`prototype` ，只有函数才有`prototype`属性，而`__proto__` 只存在于对象上。不过让人迷惑的是，`__proto__`属性在[EcmaScript 规范](https://www.ecma-international.org/ecma-262/6.0/#sec-ordinary-object-internal-methods-and-internal-slots)里叫做`[[Prototype]]`，迷不迷？
我们稍后再讲这个。 🤔

还是来看一个代码示例吧：
```
const playerFunctions = {
  setScore(newScore) {
    this.score = newScore;
  }
}

function createPlayer(userName, score) {
  const newPlayer = Object.create(playerFunctions);
  newPlayer.userName = userName;
  newPlayer.score = score;
  return newPlayer;
}

const player1 = createPlayer('sag1v', 700);
const player2 = createPlayer('sarah', 900);

console.log(player1)
console.log(player2)

```

输出：

```
player1: {
  userName: 'sag1v',
  score: 700,
  __proto__: playerFunctions
}

player2: {
  userName: 'sarah',
  score: 900,
  __proto__: playerFunctions
}

```

看到没，`player1` 和 `player2`都能访问`playerFunctions`的属性，所以都能调用`setScore`：
```
player1.setScore(1000);
player2.setScore(2000);

```

目的达到了，我们将数据和功能附加到对象上，同时也没有违背D.R.Y 原则。

但是仅仅为了创建互连对象，这么做似乎太费劲了。
1.  需要创建对象
2.  创建另一个对象用来存放功能函数
3.  用`Object.create` 将 `__proto__` 链接到那个功能函数对象
4.  设置新对象的各种属性
5.  返回这个新对象

这些工作能不能替我们完成？
### `new` 操作符，也就是构造函数

在前面的例子中我们看到，为了创建互连对象，我们在工厂方法里做了不少工作。其实 JavaScript 可以为我们完成一部分工作，只要在函数调用前用一个`new` 操作符。

不过在此之前，先确保我们对函数的理解是一致的。
#### 函数到底是什么？
```
function double(num) {
    return num * 2;
}

double.someProp = 'Hi there!';

double(5); // 10
double.someProp // Hi there!

double.prototype // {}

```

我们都知道函数是什么，对吧？我们可以声明它，然后用圆括号 `()`调用它。但是看看上面的代码，我们也可以在上面读取或创建属性，就像处理对象一样。所以我的结论是 JavaScript 中的函数不仅仅是函数，它们是“函数-对象混合体”。基本上**每个**函数都可以被调用**，并且**可以被当作对象来对待。
##### prototype 属性

原来，所有函数（[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Use_of_prototype_property)除外）都有一个`.prototype`属性。

再次提醒：
> 不是 `__proto__` 或者`[[Prototype]]`, 而是 `prototype`.

让我们再回到 **new 操作符**。
##### 执行 `new` 操作符

可用 `new` 操作符的函数大概长这样：

如果你不太确定 `this`关键字的工作原理，可以看看这篇文章 [JavaScript - The "this" key word in depth](https://www.debuggr.io/js-this-in-depth/)
```
function Player(userName, score){
  this.userName = userName;
  this.score = score;
}

Player.prototype.setScore = function(newScore){
  this.score = newScore;
}

const player1 = new Player('sag1v', 700);
const player2 = new Player('sarah', 900);

console.log(player1)
console.log(player2)

```

输出结果：

```
Player {
  userName: "sag1v",
  score: 700,
  __proto__: Player.prototype
}

Player {
  userName: "sarah",
  score: 900,
  __proto__: Player.prototype
}

```

##### 拆解代码(执行阶段)

我们使用`new` 操作符执行 `Player`函数，请注意，我将函数名从 `createPlayer` 改为`Player`，只是因为这是命名惯例。这是告诉`Player`函数的使用者，这是一个“构造函数”，应该用`new`操作符调用。

当我们使用`new`操作符调用函数时，JavaScript 为我们做了4件事:
1.  创建一个新对象
2.  把新对象赋值给 `this` 上下文
3.  把新对象的`__proto__`指向构造函数的`prototype`属性，在这里是`Player.prototype`。
4.  返回这个新对象，除非你返回一个不同的对象

如果我们把 JavaScript 自动完成的步骤写下来，大概是这样的：
```
function Player(userName, score){
  this = {} // ️ done by JavaScript
  this.__proto__ = Player.prototype // ️ done by JavaScript

  this.userName = userName;
  this.score = score;

  return this // ️ done by JavaScript
}

```

来看第3步：
> 把新对象的`__proto__`指向构造函数的`prototype`属性，在这里是`Player.prototype`

就是说我们可以在`Player.prototype`上设置任何方法，新创建的对象都自动拥有这些方法了。

我们就是这样做的：
```
Player.prototype.setScore = function(newScore){
  this.score = newScore;
}

```

这就是如何通过构造函数创建链式对象的。

顺便说一下，如果我们不使用`new`操作符，JavaScript 就不会为我们完成这些任务，我们最终只会在 `this`上下文中修改或创建一些属性。记住这一点，我们将在执行子类化时使用这个技巧。

有办法可以确保函数是通过 `new` 操作符调用的：
```
function Player(username, score){

  if(!(this instanceof Player)){
    throw new Error('Player must be called with new')
  }

  // ES2015 syntax
  if(!new.target){
    throw new Error('Player must be called with new')
  }
}

```

## 类

如果你不喜欢手写工厂方法，或者不喜欢构造函数语法，或者手动检查函数是否被`new`操作符调用，JavaScript 还提供了`class`(从ES2015开始) 语法。但是要记住，类主要是函数的语法糖，跟其他语言中的传统 `class` 非常不同，背后仍然使用**原型继承**。

来自 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) 的解释：

> JavaScript classes, introduced in ECMAScript 2015, are primarily syntactical sugar over JavaScript's existing prototype-based inheritance. The class syntax does not introduce a new object-oriented inheritance model to JavaScript.

让我们一步一步将构造函数改造成`class`：
### 声明一个类

我们使用 `class` 关键字，并将类命名为前一节中的构造函数名称。
```
class Player {

}

```

### 创建构造器

我们用前一节的构造函数的内容给我们的类创建一个`constructor` 方法：
```
class Player {
  constructor(userName, score) {
    this.userName = userName;
    this.score = score;
  }
}

```

### 添加方法

 `Player.prototype` 上定义的所有方法都可以简单地声明为类方法：
```
class Player {
  constructor(userName, score) {
    this.userName = userName;
    this.score = score;
  }

  setScore(newScore) {
    this.score = newScore;
  }
}

```

完整代码：

```
class Player {
  constructor(userName, score) {
    this.userName = userName;
    this.score = score;
  }

  setScore(newScore) {
    this.score = newScore;
  }
}

const player1 = new Player('sag1v', 700);
const player2 = new Player('sarah', 900);

console.log(player1)
console.log(player2)

```

运行下代码，结果跟之前一样：
```
Player {
  userName: "sag1v",
  score: 700,
  __proto__: Player.prototype
}

Player {
  userName: "sarah",
  score: 900,
  __proto__: Player.prototype
}

```

看到了吧，`class` 跟带有原型链的函数工作方式是一样的，只是语法不同。
## 子类（继承）

如果要定义特殊的一种`Player`，可能是氪金玩家`Player` ，拥有一些普通玩家不具备的属性，比如可以修改用户名。要怎么做呢？
So lets see what our goal here:
我们的目标是：
*   普通玩家有 `userName`，`score` 和`setScore` 方法。
*   我们想让氪金玩家拥有普通玩家所有的属性和方法外加一个`setUserName` 方法，但我们不想让普通玩家有这个功能。

在我们深入研究它之前，让我们先想象一下一串相互联系的对象：

来看代码：
```
function double(num){
    return num * 2;
}

double.toString() // where is this method coming from?

Function.prototype // {toString: f, call: f, bind: f}

double.hasOwnProperty('name') // where is this method coming from?

Function.prototype.__proto__ // -> Object.prototype {hasOwnProperty: f}

```

我们知道，如果直接在对象上找不到某个属性，JS 引擎会通过`__proto__`属性在链接对象（如果存在）上找。如果还找不到，怎么办？你应该还记得，**所有**对象都有一个 `__proto__`属性，因此会继续通过 `__proto__`属性在链接对象上找，就这么一直找下去，直到天荒地老（null 对象），基本上是`Object.prototype.__proto__`。

因此，可以这样一步一步拆解示例代码：
```
double.toString()

```

1.  `double` 没有 `toString` 方法✖️.
2.  转到 `double.__proto__`
3.  `double.__proto__` 指向 `Function.prototype` ，这个对象拥有 `toString` 方法。收工 ✔️

```
double.hasOwnProperty('name')

```

1.  `double` 没有 `hasOwnProperty` 方法 ✖️
2.  转到 `double.__proto__`
3.  `double.__proto__` 指向 `Function.prototype`。
4.  `Function.prototype` 没有 `hasOwnProperty` 方法✖️
5.  转到 `Function.prototype.__proto__`
6.  `Function.prototype.__proto__` 指向 `Object.prototype`
7.  `Object.prototype` 对象包含 `hasOwnProperty` 方法，收工✔️

下面这个小动图演示了这个过程：
![原型链示意图](/uploads/1618526-8641ac0df2544a22.webp?imageMogr2/auto-orient/strip)

现在回到创建付费用户实例的任务。我们还会继续，我们会用*OLOO模式*，*构造函数* 模式和 *类* 来实现这个特性。这样，我们将看到每个模式和特性的权衡对比。

现在让我们深入了解下继承。

## OLOO - 实现继承

采用 `OLOO` 和工厂方法模式的实现：
```
const playerFunctions = {
  setScore(newScore) {
    this.score = newScore;
  }
}

function createPlayer(userName, score) {
  const newPlayer = Object.create(playerFunctions);
  newPlayer.userName = userName;
  newPlayer.score = score;
  return newPlayer;
}

const paidPlayerFunctions = {
  setUserName(newName) {
    this.userName = newName;
  }
}

// link paidPlayerFunctions object to createPlayer object
Object.setPrototypeOf(paidPlayerFunctions, playerFunctions);

function createPaidPlayer(userName, score, balance) {
  const paidPlayer = createPlayer(name, score);
  // we need to change the pointer here
  Object.setPrototypeOf(paidPlayer, paidPlayerFunctions);
  paidPlayer.balance = balance;
  return paidPlayer
}

const player1 = createPlayer('sag1v', 700);
const paidPlayer = createPaidPlayer('sag1v', 700, 5);

console.log(player1)
console.log(paidPlayer)

```

输出结果：

```
player1 {
  userName: "sag1v",
  score: 700,
  __proto__: playerFunctions {
     setScore: ƒ
  }
}

paidPlayer {
  userName: "sarah",
  score: 900,
  balance: 5,
  __proto__: paidPlayerFunctions {
    setUserName: ƒ,
    __proto__: playerFunctions {
      setScore: ƒ
    }
  }
}

```

如你所见，我们的 `createPlayer` 函数实现没有变化，但是针对 `createPaidPlayer` 我们需要一点小技巧。

在 `createPaidPlayer` 里面我们使用 `createPlayer`创建初始对象，这样就不用重复前面的逻辑了，但不幸的是它把 `__proto__`指向了错误的对象，所以我们需要用 `Object.setPrototypeOf` 修正。

还没完，因为我们现在切断了跟 `playerFunctions` 对象的链接关系，而它上面有我们要的 `setScore`方法。这就是为什么我们还需要将`paidPlayerFunctions` 和 `playerFunctions` 关联起来，也是用 `Object.setPrototypeOf`。这样就确保 `paidPlayer`链接到了 `paidPlayerFunctions`，然后再链接到`playerFunctions`。

2层链接就需要这么多代码了，想象下如果有3层、4层该有多麻烦。

## 构造函数 - 实现继承

接下来我们用构造函数实现：
```
function Player(userName, score) {
  this.userName = userName;
  this.score = score;
}

Player.prototype.setScore = function(newScore) {
  this.score = newScore;
}

const paidPlayerFunctions = {
  setUserName(newName) {
    this.userName = newName;
  }
}

function PaidPlayer(userName, score, balance) {
  this.balance = balance;
  /* 我们不用 new 操作符 调用 "Player"，而是用 "call" 方法，这样就可以显式地传一个"this"的引用。现在"Player"函数会改变 "this"，并加上相关的属性 */
  Player.call(this, userName, score);
}

PaidPlayer.prototype.setUserName = function(newName) {
  this.userName = newName;
}

// link PaidPlayer.prototype object to Player.prototype object
Object.setPrototypeOf(PaidPlayer.prototype, Player.prototype);

const player1 = new Player('sag1v', 700);
const paidPlayer = new PaidPlayer('sarah', 900, 5);

console.log(player1)
console.log(paidPlayer)

```

结果应该跟前面的实现是一样的：
```
Player {
  userName: "sag1v",
  score: 700,
  __proto__: Player.prototype {
    setScore: ƒ
  }
}

PaidPlayer {
  userName: "sarah",
  score: 900,
  balance: 5,
  __proto__: PaidPlayer.prototype:{
    setUserName: ƒ,
    __proto__: Player.prototype {
      setScore: ƒ
    }
  }
}

```

这和我们使用工厂方法模式得到的结果是一样的，但是有些东西是由 `new` 操作符自动完成的。它可能为我们节省了几行代码，但它也带来了一些其他的麻烦。

第一个麻烦是如何使用`Player`函数来获得创建初始`Player`的逻辑。我们没有使用`new`操作符(相当违背直觉)来调用它，而是用`.call` 方法，显式地传了一个`this` 的引用，这样`Player`函数就不是作为一个构造函数来执行的，所以它不会创建一个新对象并将其赋值给`this`。
```
function PaidPlayer(userName, score, balance) {
  this.balance = balance;
  /* 我们不用 new 操作符 调用 "Player"，而是用 "call" 方法，这样就可以显式地传一个"this"的引用。现在"Player"函数会改变 "this"，并加上相关的属性 */
  Player.call(this, userName, score);
}

```

我们在这里用`Player`只是为了修改传进去的`this` ，也就是`PaidPlayer` 上下文中新创建的对象。

第二个麻烦是，将`PaidPlayer`返回的实例链接到`Player` 实例所拥有的功能上去。我们是通过 `Object.setPrototypeOf` 做到的。
```
// link PaidPlayer.prototype object to Player.prototype object
Object.setPrototypeOf(PaidPlayer.prototype, Player.prototype);

```

如你所见，引擎为我们做的事情越多，我们需要编写的代码就越少，但是随着抽象量的增长，我们就越难以跟踪底层发生了什么。
## Class - 实现继承

有了类，我们得到了更多的抽象，也就意味着更少的代码：
```
class Player {
  constructor(userName, score) {
    this.userName = userName;
    this.score = score;
  }

  setScore(newScore) {
    this.score = newScore;
  }
}

class PaidPlayer extends Player {
  constructor(userName, score, balance) {
    super(userName, score);
    this.balance = balance;
  }

  setUserName(newName) {
    this.userName = newName;
  }
}

const player1 = new Player('sag1v', 700);
const paidPlayer = new PaidPlayer('sarah', 900, 5);

console.log(player1)
console.log(paidPlayer)

```

执行结果跟使用构造函数方式是一样的：
```
Player {
  userName: "sag1v",
  score: 700,
  __proto__: Player.prototype {
    setScore: ƒ
  }
}

PaidPlayer {
  userName: "sarah",
  score: 900,
  balance: 5,
  __proto__: PaidPlayer.prototype:{
    setUserName: ƒ,
    __proto__: Player.prototype {
      setScore: ƒ
    }
  }
}

```

所以你看，类只不过是构造函数的语法糖。

当我们用 `extends` 关键字时，需要用到 `super` 函数，这是为什么？

还记得“构造函数”那一节里的这行奇怪的代码吗？
```
Player.call(this, userName, score)

```

因此`super(userName, score)` 其实是模拟了这个操作。

更准确地说，是在底层用到了 ES2015 引入的新特性：[Reflect.construct](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/construct)

引用文档里的一段话：
> The static Reflect.construct() method acts like the new operator, but as a function. It is equivalent to calling new target(...args). It gives also the added option to specify a different prototype.

所以我们不再需要 hack 构造函数了。 `super` 主要是用`Reflect.construct` 实现的。还有一点比较重要，当我们`extend`一个类时，在`constructor`中不能在执行`super()`之前使用`this`，因为这个时候`this`还没有初始化。
```
class PaidPlayer extends Player {
  constructor(userName, score, balance) {
    // "this" 还没初始化
    // super 在这里是指 Player
    super(userName, score);
    // super 底层是用 Reflect.construct 实现的
    // this = Reflect.construct(Player, [userName, score], PaidPlayer);
    this.balance = balance;
  }

  setUserName(newName) {
    this.userName = newName;
  }
}

```

## 总结

我们学习了多种方法来关联对象、封装数据和逻辑。我们知道了在JavaScript 中继承是如何工作的，通过`__proto__` 属性将对象链接到其他对象，有时还使用了多级链接。

好几个例子中我们都看到，抽象程度越高，JS 引擎为我们做的事情就越多，这也意味着更难跟踪代码背后到底发生了什么。

每一种模式都有各自的优缺点：
*   用`Object.create` ，我们需要写更多的代码，但是我们对对象有更细粒度的控制。尽管实现深度多层链接很糟心。
*   使用构造函数，我们可以通过 JavaScript 完成一些自动化的任务，但是语法可能看起来有点奇怪。我们还需要确保我们的函数通过`new`关键字调用，否则我们将面临严重的错误。深层链接也不是那么好实现。
*  通过类，我们可以获得更简洁的语法，并内置检查它是否被 `new` 操作符调用。当我们进行“继承”时，类是最好用的，我们只要用`extends` 关键字和调用`super()` ，不用像其他模式那样折腾。语法也更接近于其他语言，而且看起来很容易学。尽管这也是一个缺点，因为我们也看到了，它与其他语言中的类又不太一样，底层仍然采用“原型继承”，只是在它上面做了多层抽象。

希望这篇文章对你有所帮助，如果有任何问题和建议，欢迎关注我的微信公众号“1024译站”一起探讨。
[ ](https://dev.to/sag1v/javascript-the-prototype-chain-in-depth-2p58)
