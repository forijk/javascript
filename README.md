# JavaScript 代码规范 (整理完结)

*一种写JavaScript更合理的代码风格。*

## <a id="table-of-contents">目录</a>

1. [对象](#objects)
2. [数组](#arrays)
3. [解构](#destructuring)
4. [方法](#functions)
5. [箭头函数](#arrow-functions)
6. [类和构造器](#classes--constructors)
7. [模块](#modules)
8. [迭代器和发生器](#iterators-and-generators)
9. [属性](#properties)
10. [提升](#hoisting)
11. [比较运算符和等号](#comparison-operators--equality)
12. [控制语句](#control-statements)
13. [类型转换和强制类型转换](#type-casting--coercion)
14. [标准库](#standard-library)
15. [测试](#testing)

## <a id="objects">对象</a>

- 1.1 在创建具有动态属性名称的对象时使用计算属性名。

 > 为什么? 它允许你在一个地方定义对象的所有属性。

  ```javascript
  function getKey(k) {
    return `a key named ${k}`;
  }

  // bad
  const obj = {
    id: 5,
    name: 'San Francisco',
  };
  obj[getKey('enabled')] = true;

  // good
  const obj = {
    id: 5,
    name: 'San Francisco',
    [getKey('enabled')]: true,
  };
  ```

- 1.2 只使用引号标注无效标识符的属性。

> 为什么? 总的来说，我们认为这样更容易阅读。 它提升了语法高亮显示，并且更容易通过许多 JS 引擎优化。

```javascript
// bad
const bad = {
  'foo': 3,
  'bar': 4,
  'data-blah': 5,
};

// good
const good = {
  foo: 3,
  bar: 4,
  'data-blah': 5,
};
```

- 1.3 不能直接调用 Object.prototype 的方法，如： hasOwnProperty 、 propertyIsEnumerable 和 isPrototypeOf。

> 为什么? 这些方法可能被一下问题对象的属性追踪 - 相应的有 { hasOwnProperty: false } - 或者，对象是一个空对象 (Object.create(null))。

```javascript
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // 在模块范围内的缓存中查找一次
/* or */
import has from 'has'; // https://www.npmjs.com/package/has
// ...
console.log(has.call(object, key));
```

- 1.4 更喜欢对象扩展操作符，而不是用 Object.assign 浅拷贝一个对象。 使用对象的 rest 操作符来获得一个具有某些属性的新对象。

```javascript
// very bad
const original = { a: 1, b: 2 };
const copy = Object.assign(original, { c: 3 }); // 变异的 `original` ಠ_ಠ
delete copy.a; // 这....

// bad
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original, { c: 3 }); // copy => { a: 1, b: 2, c: 3 }

// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

const { a, ...noA } = copy; // noA => { b: 2, c: 3 }
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="arrays">数组</a>

- 2.1 使用数组展开方法 ... 来拷贝数组。

```javascript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i += 1) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

- 2.2 将一个类数组对象转换成一个数组， 使用展开方法 ... 代替 Array.from。

```javascript
const foo = document.querySelectorAll('.foo');

// good
const nodes = Array.from(foo);

// best
const nodes = [...foo];
```

- 2.3 对于对迭代器的映射，使用 Array.from 替代展开方法 ... ， 因为它避免了创建中间数组。

```javascript
// bad
const baz = [...foo].map(bar);

// good
const baz = Array.from(foo, bar);

eg.
var foo=[1,2,3];
var bar=a=>a+1;
[...foo].map(bar);
// [2, 3, 4]
Array.from(foo, bar);
// [2, 3, 4]
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="destructuring">解构</a>

- 3.1 在访问和使用对象的多个属性的时候使用对象的解构。

> 为什么? 解构可以避免为这些属性创建临时引用。

```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```

- 3.2 使用数组解构。

```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

- 3.3 对于多个返回值使用对象解构，而不是数组解构。

> 为什么? 你可以随时添加新的属性或者改变属性的顺序，而不用修改调用方。

```javascript
// bad
function processInput(input) {
  // 处理代码...
  return [left, right, top, bottom];
}

// 调用者需要考虑返回数据的顺序。
const [left, __, top] = processInput(input);

// good
function processInput(input) {
  // 处理代码...
  return { left, right, top, bottom };
}

// 调用者只选择他们需要的数据。
const { left, top } = processInput(input);
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="functions">方法</a>

- 4.1 使用命名的函数表达式代替函数声明。

> 为什么? 函数声明是挂起的，这意味着在它在文件中定义之前，很容易引用函数。这会损害可读性和可维护性。如果您发现函数的定义是大的或复杂的，以至于它干扰了对文件的其余部分的理解，那么也许是时候将它提取到它自己的模块中了!不要忘记显式地命名这个表达式，不管它的名称是否从包含变量(在现代浏览器中经常是这样，或者在使用诸如Babel之类的编译器时)。这消除了对错误的调用堆栈的任何假设。

```javascript
// bad
function foo() {
  // ...
}

// bad
const foo = function () {
  // ...
};

// good
// 从变量引用调用中区分的词汇名称
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};
```

- 4.2 Wrap立即调用函数表达式。

> 为什么? 立即调用的函数表达式是单个单元 - 包装， 并且拥有括号调用, 在括号内, 清晰的表达式。 请注意，在一个到处都是模块的世界中，您几乎不需要一个 IIFE 。

```javascript
// immediately-invoked function expression (IIFE) 立即调用的函数表达式
(function () {
  console.log('Welcome to the Internet. Please follow me.');
}());
```

- 4.3 切记不要在非功能块中声明函数 (if, while, 等)。 将函数赋值给变量。 浏览器允许你这样做，但是他们都有不同的解释，这是个坏消息。

> ECMA-262 将 block 定义为语句列表。 函数声明不是语句。

```javascript
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// good
let test;
if (currentUser) {
  test = () => {
    console.log('Yup.');
  };
}
```

- 4.4 不要使用 arguments, 选择使用 rest 语法 ... 代替。

> 为什么? ... 明确了你想要拉取什么参数。 更甚, rest 参数是一个真正的数组，而不仅仅是类数组的 arguments 。

```javascript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

- 4.5 使用默认的参数语法，而不是改变函数参数。

```javascript
// really bad
function handleThings(opts) {
  // No! We shouldn’t mutate function arguments.
  // Double bad: if opts is falsy it'll be set to an object which may
  // be what you want but it can introduce subtle bugs.
  opts = opts || {};
  // ...
}

// still bad
function handleThings(opts) {
  if (opts === void 0) {
    opts = {};
  }
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}
```

- 4.6 总是把默认参数放在最后。

```javascript
// bad
function handleThings(opts = {}, name) {
  // ...
}

// good
function handleThings(name, opts = {}) {
  // ...
}
```

- 4.7 没用变异参数。

> 为什么? 将传入的对象作为参数进行操作可能会在原始调用程序中造成不必要的变量副作用。

```javascript
// bad
function f1(obj) {
  obj.key = 1;
}

// good
function f2(obj) {
  const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
}
```

- 4.8 不要再赋值参数。

> 为什么? 重新赋值参数会导致意外的行为，尤其是在访问 `arguments` 对象的时候。 它还可能导致性能优化问题，尤其是在 V8 中。

```javascript
// bad
function f1(a) {
  a = 1;
  // ...
}

function f2(a) {
  if (!a) { a = 1; }
  // ...
}

// good
function f3(a) {
  const b = a || 1;
  // ...
}

function f4(a = 1) {
  // ...
}
```
- 4.9 优先使用扩展运算符 ... 来调用可变参数函数。

> 为什么? 它更加干净，你不需要提供上下文，并且你不能轻易的使用 `apply` 来 `new` 。

```javascript
// bad
const x = [1, 2, 3, 4, 5];
console.log.apply(console, x);

// good
const x = [1, 2, 3, 4, 5];
console.log(...x);

// bad
new (Function.prototype.bind.apply(Date, [null, 2016, 8, 5]));

// good
new Date(...[2016, 8, 5]);
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="arrow-functions">箭头函数</a>

- 5.1 避免箭头函数符号 (=>) 和比较运算符 (<=, >=) 的混淆。

```javascript
// bad
const itemHeight = item => item.height > 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) => item.height > 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = item => (item.height > 256 ? item.largeSize : item.smallSize);

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item;
  return height > 256 ? largeSize : smallSize;
};
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="classes--constructors">类和构造器</a>

- 6.1 尽量使用 class. 避免直接操作 prototype .

> 为什么? `class` 语法更简洁，更容易推理。

```javascript
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}
Queue.prototype.pop = function () {
  const value = this.queue[0];
  this.queue.splice(0, 1);
  return value;
};

// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
```

- 6.2 使用 extends 来扩展继承。

> 为什么? 它是一个内置的方法，可以在不破坏 `instanceof` 的情况下继承原型功能。

```javascript
// 彻底搞懂JS中的prototype、__proto__与constructor（图解）

// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}
inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function () {
  return this.queue[0];
};

// good
class PeekableQueue extends Queue {
  peek() {
    return this.queue[0];
  }
}
```

- 6.3 方法返回了 `this` 来供其内部方法调用。

```javascript
// bad
Jedi.prototype.jump = function () {
  this.jumping = true;
  return true;
};

Jedi.prototype.setHeight = function (height) {
  this.height = height;
};

const luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined

// good
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump()
  .setHeight(20);
```

- 6.4 只要在确保能正常工作并且不产生任何副作用的情况下，编写一个自定义的 toString() 方法也是可以的。

```javascript
class Jedi {
  constructor(options = {}) {
    this.name = options.name || 'no name';
  }

  getName() {
    return this.name;
  }

  toString() {
    return `Jedi - ${this.getName()}`;
  }
}
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="modules">模块</a>

- 7.1 你可能经常使用模块 (import/export) 在一些非标准模块的系统上。 你也可以在你喜欢的模块系统上相互转换。

> 为什么? 模块是未来的趋势，让我们拥抱未来。

```javascript
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ok
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

- 7.2 只从一个路径导入所有需要的东西。

> 为什么? 从同一个路径导入多个行，使代码更难以维护。

```javascript
// bad
import foo from 'foo';
// … 其他导入 … //
import { named1, named2 } from 'foo';

// good
import foo, { named1, named2 } from 'foo';

// good
import foo, {
  named1,
  named2,
} from 'foo';
```

- 7.3 不要导出可变的引用。

> 为什么? 在一般情况下，应该避免发生突变，但是在导出可变引用时及其容易发生突变。虽然在某些特殊情况下，可能需要这样，但是一般情况下只需要导出常量引用。

```javascript
// bad
let foo = 3;
export { foo };

// good
const foo = 3;
export { foo };
```

- 7.4 在单个导出的模块中，选择默认模块而不是指定的导出。

> 为什么? 为了鼓励更多的文件只导出一件东西，这样可读性和可维护性更好。

```javascript
// bad
export function foo() {}

// good
export default function foo() {}
```

- 7.5 将所有的 imports 语句放在所有非导入语句的上边。

> 为什么? 由于所有的 `import` 都被提前，保持他们在顶部是为了防止意外发生。

```javascript
// bad
import foo from 'foo';
foo.init();

import bar from 'bar';

// good
import foo from 'foo';
import bar from 'bar';

foo.init();
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="iterators-and-generators">迭代器和发生器</a>

- 8.1 不要使用迭代器。 你应该使用 JavaScript 的高阶函数代替 for-in 或者 for-of。

> 为什么? 这是我们强制的规则。 拥有返回值得纯函数比这个更容易解释。

> 使用 `map()` / `every()` / `filter()` / `find()` / `findIndex()` / `reduce()` / `some()` / ... 遍历数组， 和使用 `Object.keys()` / `Object.values()` / `Object.entries()` 迭代你的对象生成数组。

```javascript
const numbers = [1, 2, 3, 4, 5];

// bad
let sum = 0;
for (let num of numbers) {
  sum += num;
}
sum === 15;

// good
let sum = 0;
numbers.forEach((num) => {
  sum += num;
});
sum === 15;

// best (use the functional force)
const sum = numbers.reduce((total, num) => total + num, 0);
sum === 15;

// bad
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// good
const increasedByOne = [];
numbers.forEach((num) => {
  increasedByOne.push(num + 1);
});

// best (keeping it functional)
const increasedByOne = numbers.map(num => num + 1);
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="properties">属性</a>

- 9.1 访问属性时使用点符号。

```javascript
const luke = {
  jedi: true,
  age: 28,
};

// bad
const isJedi = luke['jedi'];

// good
const isJedi = luke.jedi;
```

- 9.2 使用变量访问属性时，使用 []表示法。

```javascript
const luke = {
  jedi: true,
  age: 28,
};

function getProp(prop) {
  return luke[prop];
}

const isJedi = getProp('jedi');
```

- 9.3 计算指数时，可以使用 ** 运算符。 

```javascript
// bad
const binary = Math.pow(2, 10);

// good
const binary = 2 ** 10;
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="hoisting">提升</a>

- 10.1 var 定义的变量会被提升到函数范围的最顶部，但是它的赋值不会。const 和 let 声明的变量受到一个称之为 Temporal Dead Zones (TDZ) 的新概念保护。 知道为什么 typeof 不再安全 是很重要的。

```javascript
// 我们知道这个行不通 (假设没有未定义的全局变量)
function example() {
  console.log(notDefined); // => throws a ReferenceError
}

// 在引用变量后创建变量声明将会因变量提升而起作用。
// 注意: 真正的值 `true` 不会被提升。
function example() {
  console.log(declaredButNotAssigned); // => undefined
  var declaredButNotAssigned = true;
}

// 解释器将变量提升到函数的顶部
// 这意味着我们可以将上边的例子重写为：
function example() {
  let declaredButNotAssigned;
  console.log(declaredButNotAssigned); // => undefined
  declaredButNotAssigned = true;
}

// 使用 const 和 let
function example() {
  console.log(declaredButNotAssigned); // => throws a ReferenceError
  console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
  const declaredButNotAssigned = true;
}
```

- 10.2 匿名函数表达式提升变量名，而不是函数赋值。

```javascript
function example() {
  console.log(anonymous); // => undefined

  anonymous(); // => TypeError anonymous is not a function

  var anonymous = function () {
    console.log('anonymous function expression');
  };
}
```

- 10.3 命名函数表达式提升的是变量名，而不是函数名或者函数体。

```javascript
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  superPower(); // => ReferenceError superPower is not defined

  var named = function superPower() {
    console.log('Flying');
  };
}

// 当函数名和变量名相同时也是如此。
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  var named = function named() {
    console.log('named');
  };
}
```

- 10.4 函数声明提升其名称和函数体。

```javascript
function example() {
  superPower(); // => Flying

  function superPower() {
    console.log('Flying');
  }
}
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="comparison-operators--equality">比较运算符和等号</a>

- 11.1 使用 === 和 !== 而不是 == 和 !=。 

- 11.2 条件语句，例如 if 语句使用 ToBoolean 的抽象方法来计算表达式的结果，并始终遵循以下简单的规则：
  - **Objects** 的取值为： **true**
  - **Undefined** 的取值为： **false**
  - **Null** 的取值为： **false**
  - **Booleans** 的取值为： **布尔值的取值**
  - **Numbers** 的取值为：如果为 **+0, -0, or NaN** 值为 **false** 否则为 **true**
  - **Strings** 的取值为: 如果是一个空字符串 `''` 值为 **false** 否则为 **true**

```javascript
if ([0] && []) {
  // true
  // 一个数组（即使是空的）是一个对象，对象的取值为 true
}
```

- 11.3 对于布尔值使用简写，但是对于字符串和数字进行显式比较。

```javascript
// bad
if (isValid === true) {
  // ...
}

// good
if (isValid) {
  // ...
}

// bad
if (name) {
  // ...
}

// good
if (name !== '') {
  // ...
}

// bad
if (collection.length) {
  // ...
}

// good
if (collection.length > 0) {
  // ...
}
```

- 11.4 在 case 和 default 的子句中，如果存在声明 (例如. let, const, function, 和 class)，使用大括号来创建块 。

> 为什么? 语法声明在整个 `switch` 块中都是可见的，但是只有在赋值的时候才会被初始化，这种情况只有在 `case` 条件达到才会发生。 当多个 `case` 语句定义相同的东西是，这会导致问题问题。

```javascript
// bad
switch (foo) {
  case 1:
    let x = 1;
    break;
  case 2:
    const y = 2;
    break;
  case 3:
    function f() {
      // ...
    }
    break;
  default:
    class C {}
}

// good
switch (foo) {
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    const y = 2;
    break;
  }
  case 3: {
    function f() {
      // ...
    }
    break;
  }
  case 4:
    bar();
    break;
  default: {
    class C {}
  }
}
```

- 11.5 三目表达式不应该嵌套，通常是单行表达式。 

```javascript
// bad
const foo = maybe1 > maybe2
  ? "bar"
  : value1 > value2 ? "baz" : null;

// 分离为两个三目表达式
const maybeNull = value1 > value2 ? 'baz' : null;

// better
const foo = maybe1 > maybe2
  ? 'bar'
  : maybeNull;

// best
const foo = maybe1 > maybe2 ? 'bar' : maybeNull;
```

- 11.6 避免不必要的三目表达式。

```javascript
// bad
const foo = a ? a : b;
const bar = c ? true : false;
const baz = c ? false : true;

// good
const foo = a || b;
const bar = !!c;
const baz = !c;
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="control-statements">控制语句</a>

- 12.1 如果你的控制语句 (if, while 等) 太长或者超过了一行最大长度的限制，则可以将每个条件（或组）放入一个新的行。 逻辑运算符应该在行的开始。

> 为什么? 要求操作符在行的开始保持对齐并遵循类似方法衔接的模式。 这提高了可读性，并且使更复杂的逻辑更容易直观的被理解。

```javascript
// bad
if ((foo === 123 || bar === 'abc') && doesItLookGoodWhenItBecomesThatLong() && isThisReallyHappening()) {
  thing1();
}

// bad
if (foo === 123 &&
  bar === 'abc') {
  thing1();
}

// bad
if (foo === 123
  && bar === 'abc') {
  thing1();
}

// bad
if (
  foo === 123 &&
  bar === 'abc'
) {
  thing1();
}

// good
if (
  foo === 123
  && bar === 'abc'
) {
  thing1();
}

// good
if (
  (foo === 123 || bar === 'abc')
  && doesItLookGoodWhenItBecomesThatLong()
  && isThisReallyHappening()
) {
  thing1();
}

// good
if (foo === 123 && bar === 'abc') {
  thing1();
}
```

- 12.2 不要使用选择操作符代替控制语句。

```javascript
// bad
!isRunning && startRunning();

// FIXME: 两种注释
// TODO: 两种注释

// good
if (!isRunning) {
  startRunning();
}
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="type-casting--coercion">类型转换和强制类型转换</a>

- 13.1 字符类型：

```javascript
// => this.reviewScore = 9;

// bad
const totalScore = new String(this.reviewScore); // typeof totalScore is "object" not "string"

// bad
const totalScore = this.reviewScore + ''; // invokes this.reviewScore.valueOf()

// bad
const totalScore = this.reviewScore.toString(); // isn’t guaranteed to return a string

// good
const totalScore = String(this.reviewScore);
```

- 13.2 数字类型：使用 Number 进行类型铸造和 parseInt 总是通过一个基数来解析一个字符串。

```javascript
const inputValue = '4';

// bad
const val = new Number(inputValue);

// bad
const val = +inputValue;

// bad
const val = inputValue >> 0;

// bad
const val = parseInt(inputValue);

// good
const val = Number(inputValue);

// good
const val = parseInt(inputValue, 10);
```

- 13.3 如果出于某种原因，你正在做一些疯狂的事情，而 parseInt 是你的瓶颈，并且出于 性能问题 需要使用位运算， 请写下注释，说明为什么这样做和你做了什么。

```javascript
// good
/**
  * parseInt 使我的代码变慢。
  * 位运算将一个字符串转换成数字更快。
  */
const val = inputValue >> 0;
```

- 13.4 注意： 当你使用位运算的时候要小心。 数字总是被以 64-bit 值 的形式表示，但是位运算总是返回一个 32-bit 的整数 (来源)。 对于大于 32 位的整数值，位运算可能会导致意外行为。讨论。 最大的 32 位整数是： 2,147,483,647。

```javascript
2147483647 >> 0; // => 2147483647
2147483648 >> 0; // => -2147483648
2147483649 >> 0; // => -2147483647
```

- 13.5 布尔类型：

```javascript
const age = 0;

// bad
const hasAge = new Boolean(age);

// good
const hasAge = Boolean(age);

// best
const hasAge = !!age;
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="standard-library">标准库</a>

- 14.1 标准库 包含功能已损坏的实用工具，但因为遗留原因而保留。

- 14.2 使用 Number.isNaN 代替全局的 isNaN

> 为什么? 全局的 `isNaN` 强制非数字转化为数字，对任何强制转化为 NaN 的东西都返回 true。

> 如果需要这种行为，请明确说明。

```javascript
// bad
isNaN('1.2'); // false
isNaN('1.2.3'); // true

// good
Number.isNaN('1.2.3'); // false
Number.isNaN(Number('1.2.3')); // true
```

- 14.3 使用 Number.isFinite 代替全局的 isFinite

> 为什么? 全局的 `isFinite` 强制非数字转化为数字，对任何强制转化为有限数字的东西都返回 true。

> 如果需要这种行为，请明确说明。

```javascript
// bad
isFinite('2e3'); // true

// good
Number.isFinite('2e3'); // false
Number.isFinite(parseInt('2e3', 10)); // true
```

**[⬆ 返回目录](#table-of-contents)**

## <a id="Testing">测试</a>

- 15.1 **没有，但是认真**:

  - 无论你使用那种测试框架，都应该编写测试！
  - 努力写出许多小的纯函数，并尽量减少发生错误的地方。
  - 对于静态方法和 mock 要小心----它们会使你的测试更加脆弱。
  - 我们主要在 Airbnb 上使用 [`mocha`](https://www.npmjs.com/package/mocha) 和 [`jest`](https://www.npmjs.com/package/jest) 。 [`tape`](https://www.npmjs.com/package/tape) 也会用在一些小的独立模块上。
  - 100%的测试覆盖率是一个很好的目标，即使它并不总是可行的。
  - 无论何时修复bug，都要编写一个回归测试。在没有回归测试的情况下修复的bug在将来几乎肯定会再次崩溃。

**[⬆ 返回目录](#table-of-contents)**

### 完结~ 感谢 Airbnb ~
