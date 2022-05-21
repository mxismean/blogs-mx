# ES7 到 ES12 常用新语法

> **summary**: ES7、ES8、ES9、ES10、ES11、ES12 常用新语法
>
> **tags**: 前端、JavaScript
>
> **author**: 大熊

---

众所周知，ECMAScript 的迭代是很快的，想必作为前端开发人员对近几年 ES6 的新语法已经十分熟悉了，但是 ES7 到 ES12 中一些新增的 ECMAScript 提案，可能还是没有广泛地被开发者所熟知。本文带着大家一起来了解一下 2016 到 2021 年新增的一些 ECMAScript 语法以及提案，帮助大家更好地应用于自己的项目中。

> **TC39 规范介绍：**
>
> `Stage 0`（strawman），任何 TC39 的成员都可以提交。
> 
> `Stage 1`（proposal），进入此阶段意味着这一提案被认为是正式的了，需要对此提案的场景与 API 进行详尽的描述。
>
> `Stage 2`（draft），这一阶段的提案如果能最终进入到标准，那么在之后的阶段都不会有太大的变化，因为理论上只接受增量修改。
> 
> `Stage 3`（candidate），这一阶段的提案只有在遇到了重大问题才会修改，规范文档需要被全面的完成。
> 
> `Stage 4`（finished），这一阶段的提案将会被纳入到 ES 每年发布的规范之中。

## ES7（ES2016）

### 1、Array.prototype.includes

#### 概述

> `includes()` 方法用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 `true`，否则返回 `false`。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-Array.prototype.includes)

#### 动机

使用 `ECMAScript` 数组时，通常需要确定数组是否包含某个元素。普遍的方法是：

```javascript
if (arr.indexOf(el) !== -1) {
  ...
}
```

严格意义来说，使用 `indexOf` 来确定数组中是否包含某个元素是存在问题的：

- 语义不符，我们的目的是想知道数组中是否包含某个元素，并不是想知道数组中某个元素首次出现的索引。
- 对 `NaN` 不生效，`[NaN].indexOf(NaN) === -1`。

#### 语法

> arr.includes(valueToFind[, fromIndex])
> 
> - `valueToFind`：需要查找的元素值。
> - `fromIndex`：可选的。从 `fromIndex` 索引处开始查找 `valueToFind`。如果为负值，则按升序从 `arr.length + fromIndex` 的索引开始搜（即从末尾开始往前跳 `fromIndex` 的绝对值个索引，然后往后搜寻）。默认为 `0`。

**例子：**

```javascript
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true
```

#### 注意事项

1、如果 `fromIndex` 大于等于数组的长度，则会返回 `false`，且该数组不会被搜索。

```javascript
const arr = ['a', 'b', 'c'];
arr.includes('c', 3);   // false
arr.includes('c', 100); // false
```

2、如果 `fromIndex` 为负值，计算出的索引将作为开始搜索 `valueToFind` 的位置。如果计算出的索引小于 `0`，则整个数组都会被搜索。

```javascript
// array length is 3
// fromIndex is -100
// computed index is 3 + (-100) = -97

const arr = ['a', 'b', 'c'];

arr.includes('a', -100); // true
arr.includes('b', -100); // true
arr.includes('c', -100); // true
arr.includes('a', -2);   // false
```

3、`includes()` 方法有意设计为通用方法。它不要求 `this` 值是数组对象，所以它可以被用于其他类型的对象（比如类数组对象）。下面的例子展示了在函数的 `arguments` 对象上调用 `includes()` 方法。

```javascript
(function() {
  console.log([].includes.call(arguments, 'a')); // true
  console.log([].includes.call(arguments, 'd')); // false
})('a', 'b', 'c');
```

### 2、Exponentiation Operator

#### 概述

> 求幂运算符（`**`）返回将第一个操作数加到第二个操作数的幂的结果。它等效于 `Math.pow`，不同之处在于它也接受 `BigInt` 作为操作数。
> 
> 该提案目前处于 Stage 3 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-exponentiation-operator)

#### 语法

> Operator: `var1 ** var2`

求幂运算符是是右结合的: `a ** b ** c` 等于 `a ** (b ** c)`。

**例子：**

```javascript
2 ** 3   // 8
3 ** 2   // 9
3 ** 2.5 // 15.588457268119896
10 ** -1 // 0.1
NaN ** 2 // NaN

2 ** 3 ** 2   // 512
2 ** (3 ** 2) // 512
(2 ** 3) ** 2 // 64

-(2 ** 2) // -4

(-2) ** 2 // 4
```

## ES8（ES2017）

### 1、async 和 await

#### 概述

> `async` 和 `await` 关键字是基于 `Generator` 函数的语法糖，使异步代码更易于编写和阅读。通过使用它们，异步代码看起来更像是老式同步代码。
> 
> [TC39 直通车](https://github.com/tc39/proposal-async-await)

#### 动机

简化异步编程写法。

使用 Generator 函数，依次读取两个文件：

```javascript
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

```

上面代码的函数 gen 可以写成 async 函数，就是下面这样：

```javascript
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

一比较就会发现，`async` 函数就是将 `Generator` 函数的星号（`*`）替换成 `async`，将 `yield` 替换成 `await`。

`async` 函数对 `Generator` 函数的改进，体现在以下四点：

- 内置执行器。Generator 函数的执行必须靠执行器，所以才有了 `co` 模块，而 `async` 函数自带执行器。也就是说，`async` 函数的执行，与普通函数一模一样，只要一行。
- 更好的语义。`async` 和 `await`，比起星号（`*`）和 `yield`，语义更清楚了。`async` 表示函数里有异步操作，`await` 表示紧跟在后面的表达式需要等待结果。
- 更广的适用性。`co` 模块约定，`yield` 命令后面只能是 `Thunk` 函数或 `Promise` 对象，而 `async` 函数的 `await` 命令后面，可以是 `Promise` 对象和原始类型的值（数值、字符串和布尔值等，但这时会自动转成立即 `resolved` 的 `Promise` 对象）。
- 返回值是 Promise。`async` 函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用 `then` 方法指定下一步的操作。进一步说，`async` 函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而 `await` 命令就是内部 `then` 命令的语法糖。

#### 语法

```javascript
async function getData() {
  const res = await api.getTableData(); // await 异步任务
  // do something  
}
```

**例子：** 

```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

### 2、Object.values 和 Object.entries

#### 概述

> `Object.values()` 方法返回一个给定对象自身的所有可枚举属性值的数组，值的顺序与使用 for...in 循环的顺序相同（区别在于 for...in 循环枚举原型链中的属性）。
> 
> `Object.entries()` 方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in 循环遍历该对象时返回的顺序一致（区别在于 for...in 循环还会枚举原型链中的属性）。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-object-values-entries)

#### 语法

> Object.values(obj)
> - obj：被返回可枚举属性值的对象。
> 
> Object.entries(obj)
> - obj：可以返回其可枚举属性的键值对的对象。

**例子：**

```javascript
Object.values({a: 1, b: 2, c: 3});  // [1, 2, 3]
Object.keys({a: 1, b: 2, c: 3});    // [a, b, c]
Object.entries({a: 1, b: 2, c: 3}); // [["a", 1], ["b", 2], ["c", 3]]
```

### 3、Object.getOwnPropertyDescriptors

#### 概述

> 获取一个对象所有自身属性的描述符，如果没有任何自身属性，则返回空对象。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-object-getownpropertydescriptors)

#### 动机

ECMAScript 中没有一个方法能够简化两个对象之间的拷贝。如今，函数式编程和不可变对象是复杂应用程序的重要组成部分，每个框架或库都在实现自己的样板文件，以便在组合对象或原型之间正确复制属性。

回到 `Object.assign`，它大多数情况下会出现一些不希望出现的行为，因为它的复制方式会吞噬行为：它直接访问属性和符号，而不是它们的描述符，丢弃可能的访问器，在组合更复杂的对象或类的原型时可能会导致危险。

检索所有描述符，无论是否可枚举，也是实现类及其原型组合的关键，因为默认情况下，它们具有不可枚举的方法和访问器。

此外，装饰器可以轻松地从一个类或 mixin 中获取所有描述符，并通过 `Object.defineProperties` 过滤不需要的描述符。

`Object.assign` 只适合做对象的浅拷贝。

#### 语法

> Object.getOwnPropertyDescriptors(obj)
> 
> - obj：任意对象

**例子：**

```javascript
const a = { 
  b: 123,
  c: '文字内容',
  d() { return 2; }
};
Object.getOwnPropertyDescriptors(a);
// 输出结果
{
  b:{
    configurable: true,
    enumerable: true,
    value: 123,
    writable: true,
    [[Prototype]]: Object,
  }
  c:{
    configurable: true,
    enumerable: true,
    value: "文字内容",
    writable: true,
    [[Prototype]]: Object,
  }
  d:{
    configurable: true,
    enumerable: true,
    value: ƒ d(),
    writable: true,
    [[Prototype]]: Object,
  },
  [[Prototype]]: Object,
}
```

### 4、String.prototype.padStart 和 String.prototype.padEnd

#### 概述

> `padStart()` 方法用另一个字符串填充当前字符串（如果需要的话，会重复多次），以便产生的字符串达到给定的长度。从当前字符串的开头（左侧）开始填充。
> 
> `padEnd()` 方法会用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-string-pad-start-end)

#### 语法

> str.padStart(targetLength [, padString])
> 
> - `targetLength`：当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。
> - `padString`：可选的。填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。此参数的默认值为 `" "`（U+0020）。
> 
> str.padEnd(targetLength [, padString])
> - `targetLength`：当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。
> - `padString`：可选的。填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。此参数的缺省值为 `" "`（U+0020）。

**例子：**

```javascript
'abc'.padStart(10);         // "       abc"
'abc'.padStart(10, "foo");  // "foofoofabc"
'abc'.padStart(6,"123465"); // "123abc"
'abc'.padStart(8, "0");     // "00000abc"
'abc'.padStart(1);          // "abc"
```

```javascript
'abc'.padEnd(10);          // "abc       "
'abc'.padEnd(10, "foo");   // "abcfoofoof"
'abc'.padEnd(6, "123456"); // "abc123"
'abc'.padEnd(1);           // "abc"
```

## ES9（ES2018）

### 1、for await...of

#### 概述

> `for await...of` 语句创建一个循环，该循环遍历异步可迭代对象以及同步可迭代对象，包括: 内置的 String、Array、类似数组对象（例如 arguments 或 NodeList）、TypedArray、Map、Set 和用户定义的异步/同步迭代器。它使用对象的每个不同属性的值调用要执行的语句来调用自定义迭代钩子。
> 
> 类似于 `await` 运算符一样，该语句只能在一个 `async function` 内部使用。
> 
> [TC39 直通车](https://github.com/tc39/proposal-async-iteration)

#### 动机

迭代器接口（在 ECMAScript 2015 中引入）是一种顺序数据访问协议，它支持开发通用和可组合的数据使用者和转换器。
由于在迭代器方法返回时必须知道序列中的下一个值和数据源的“完成”状态，因此迭代器仅适用于表示同步数据源。虽然 JavaScript 程序员遇到的许多数据源是同步的（例如内存列表和其他数据结构），但许多其他数据源不是。例如，任何需要 I/O 访问的数据源通常会使用基于事件或流式异步 API 来表示。不幸的是，迭代器不能用于表示这样的数据源。

（即使是 promise 的迭代器也是不够的，因为它只允许异步确定值，但需要同步确定“完成”状态。）

为了为异步数据源提供通用的数据访问协议，因此引入了异步迭代语句 `for await...of`。


#### 语法

> async function process(iterable) {
>   for await (variable of iterable) {
>     // Do something.
>   }
> }
> 
> - `variable`：在每次迭代中，将不同属性的值分配给变量。变量有可能以 const、let 或者 var 来声明。
> - `iterable`：被迭代枚举其属性的对象。与 for...of 相比，这里的对象可以返回 Promise，如果是这样，那么 variable 将是 Promise 所包含的值，否则是值本身。

**例子：**

```javascript
function getTime(seconds){
  return new Promise(resolve=>{
    setTimeout(() => {
      resolve(seconds)
    }, seconds);
  })
}
async function test(){
  let arr = [getTime(500), getTime(300), getTime(3000)];
  for await (let x of arr){
    console.log(x); // 500  300 3000 按顺序的
  }
}

test()
```

### 2、Promise.prototype.finally

#### 概述

> `finally()` 方法返回一个 Promise。在 Promise 结束时，无论结果是 fulfilled 还是 rejected，都会执行指定的回调函数。这为在 Promise 是否成功完成后都需要执行的代码提供了一种方式。
> 
> 这避免了同样的语句需要在 `then()` 和 `catch()` 中各写一次的情况。
> 
> [TC39 直通车](https://github.com/tc39/proposal-promise-finally)

#### 动机

有些时候在 Promise 完成时，不管结果是 `fulfilled` 还是 `rejected`，我们想执行一些清理工作，或是设置一些变量，这个是时候，我们不得不在 `resolve` 和 `catch` 中把同样的代码书写两遍，这是重复且无意义的工作。
`Promise.finally()` 提案，可以在 Promise 完成时，无论结果是 `fulfilled` 还是 `rejected`，都会执行 `finally` 函数。

#### 语法

> Promise.resolve().then().catch(e => e).finally(onFinally);
> 
> - `onFinally`：Promise 结束后调用的 Function。

**例子：**

```javascript
let isLoading = true;
fetch(myRequest).then(function(response) {
  const contentType = response.headers.get("content-type");
  if(contentType && contentType.includes("application/json")) {
    return response.json();
  }
  throw new TypeError("Oops, we haven't got JSON!");
})
.then(function(json) { /* process your JSON further */ })
.catch(function(error) { console.log(error); })
.finally(function() { isLoading = false; });
```

#### 注意事项

`finally()` 虽然与 `.then(onFinally, onFinally)` 类似，但它们不同的是：

- 调用内联函数时，不需要多次声明该函数或为该函数创建一个变量保存它。
- 由于无法知道 promise 的最终状态，所以 finally 的回调函数中不接收任何参数，它仅用于无论最终结果如何都要执行的情况。
- 与 `Promise.resolve(2).then(() => {}, () => {})`（resolved 的结果为 undefined）不同，`Promise.resolve(2).finally(() => {})` resolved 的结果为 2。
- 同样，`Promise.reject(3).then(() => {}, () => {})`（fulfilled 的结果为 undefined），`Promise.reject(3).finally(() => {})` rejected 的结果为 3。

### 3、Object Rest/Spread Properties

#### 概述

> 将 `Rest` 语法添加到解构中。`Rest` 属性收集那些尚未被解构模式拾取的剩余可枚举属性键。
> 
> 将 `Spread` 语法添加到对象字面量中。将已有对象的所有可枚举（ `enumerable` ）属性拷贝到新构造的对象中。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-object-rest-spread)

#### 动机

浅拷贝（Shallow-cloning，不包含 prototype）和对象合并，可以使用更简短的 `Spread` 语法。而不必再使用 `Object.assign()` 方式。

```javascript
const obj1 = { foo: 'bar', x: 42 };
const obj2 = { foo: 'baz', y: 13 };

const clonedObj = { ...obj1 };
// 克隆后的对象: { foo: "bar", x: 42 }

const mergedObj = { ...obj1, ...obj2 };
// 合并后的对象: { foo: "baz", x: 42, y: 13 }
```

有些时候我们在解构一个对象时，可能只关心它的其中一两个属性，这个时候 `Rest` 语法可以极大简化变量提取过程。

```javascript
const { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x; // 1
y; // 2
```

#### 语法

Rest

```javascript
const { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x; // 1
y; // 2
z; // { a: 3, b: 4 }
```

Spread

```javascript
let n = { x, y, ...z };
n; // { x: 1, y: 2, a: 3, b: 4 }
```

#### 注意事项

`Object.assign()` 函数会触发 `setters`，而 `Spread` 语法则不会。

## ES10（ES2019）

### 1、Array.prototype.flat 和 Array.prototype.flatMap

#### 概述

> `flat()` 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。
> 
> `flatMap()` 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。
> 
> `flatMap` 方法与 `map` 方法结合深度 `depth` 为 `1` 的 `flat` 几乎相同。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-flatMap)

#### 动机

如果我们想扁平化一个多层嵌套的数组，代码如下：

```javascript
// 使用 reduce、concat 和递归展开无限多层嵌套的数组
const arr = [1, 2, 3, [1, 2, 3, 4, [2, 3, 4]]];

function flatDeep(arr, d = 1) {
  return d > 0 ? arr.reduce((acc, val) => acc.concat(Array.isArray(val) ? flatDeep(val, d - 1) : val), []) : arr.slice();
};

flatDeep(arr, Infinity);
// [1, 2, 3, 1, 2, 3, 4, 2, 3, 4]
```

`flat` 提案正是用来简化这个流程：

```javascript
const arr = [1, 2, 3, [1, 2, 3, 4, [2, 3, 4]]];
arr.flat(Infinity);
// [1, 2, 3, 1, 2, 3, 4, 2, 3, 4]
```

如果我们想将包含几句话的数组拆分成单个词组成的新数组，代码如下：

```javascript
const arr = ["it's Sunny in", "", "California"];
const arr1 = arr.map(x => x.split(" "));
// [["it's","Sunny","in"],[""],["California"]];
arr1.flat(2);
// ["it's", "Sunny", "in", "", "California"]
```

`flatMap` 提案可以用来简化该流程：

```javascript
const arr = ["it's Sunny in", "", "California"];
arr.flatMap(x => x.split(" "));
// ["it's", "Sunny", "in", "", "California"]
```

#### 语法

> arr.flat([depth])
> 
> - `depth`：可选的。指定要提取嵌套数组的结构深度，默认值为 `1`。

**例子：**

```javascript
[1, 2, [3, 4]].flat(1); 
// [1, 2, 3, 4]

[1, 2, [3, 4, [5, 6]]].flat(2); 
// [1, 2, 3, 4, 5, 6]

[1, 2, [3, 4, [5, 6]]].flat(Infinity); 
// [1, 2, 3, 4, 5, 6]
```

> arr.flatMap(
>   function callback(
>     currentValue[, index[, array]]) {
>     // return element for new_array
> }[, thisArg])
> 
> - `callback`：可以生成一个新数组中的元素的函数，可以传入三个参数：
>   - `currentValue`：当前正在数组中处理的元素。
>   - `index`：可选的。数组中正在处理的当前元素的索引。
>   - `array`：可选的。数组中正在处理的当前元素的索引。
> - `thisArg`：可选的。执行 callback 函数时 使用的 `this` 值。

**例子：**

```javascript
[1, 2, 3, 4].flatMap(a => [a ** 2]); 
// [1, 4, 9, 16]
```

`map()` 与 `flatMap()`

```javascript
const arr1 = [1, 2, 3, 4];

arr1.map(x => [x * 2]);
// [[2], [4], [6], [8]]

arr1.flatMap(x => [x * 2]);
// [2, 4, 6, 8]

arr1.map(x => [x * 2]).flat(1);
// [2, 4, 6, 8]

// only one level is flattened
arr1.flatMap(x => [[x * 2]]);
// [[2], [4], [6], [8]]
```

```javascript
let arr1 = ["it's Sunny in", "", "California"];

arr1.map(x => x.split(" "));
// [["it's","Sunny","in"],[""],["California"]]

arr1.flatMap(x => x.split(" "));
// ["it's","Sunny","in", "", "California"]
```

### 2、Object.fromEntries

#### 概述

> `Object.fromEntries()` 方法把键值对列表转换为一个对象。
> 
> `Object.fromEntries()` 执行与 `Object.entries` 互逆的操作。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-object-from-entries)

#### 动机

```javascript
const obj = { foo: true, bar: false };
const map = new Map(Object.entries(obj));
```

如果要将类似上面代码中的 `map` 再转换为 `obj`，我们不得不编写帮助函数：

```javascript
const obj = Array.from(map).reduce((acc, [key, val]) => Object.assign(acc, { [key]: val }), {});
```

`Object.fromEntries` 提案正是用来简化这个流程。

#### 语法

> Object.fromEntries(iterable);
> 
> - `iterable`：类似 Array、Map 或者其它实现了可迭代协议的可迭代对象。

**例子：**

嵌套数组

```javascript
const obj = Object.fromEntries([['a', 0], ['b', 1]]); 
// { a: 0, b: 1 }
```

Object.entries

```javascript
const obj = { abc: 1, def: 2, ghij: 3 };
const res = Object.fromEntries(
  Object.entries(obj)
  .filter(([key, val]) => key.length === 3)
  .map(([key, val]) => [key, val * 2])
);
// { abc: 2, def: 4 }
```

通过 `Object.fromEntries`，可以将 `Map` 转化为 `Object`：

```javascript
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
const obj = Object.fromEntries(map);
// { a: 1, b: 2, c: 3 }
```

### 3、String.prototype.trimStart 和 String.prototype.trimEnd

#### 概述

> 去除字符串首尾的连续空白符。
> 
> `trimStart()` 方法从字符串的开头删除空白符。`trimLeft()` 是此方法的别名。
> 
> `trimEnd()` 方法从字符串的结尾删除空白符。`trimRight()` 是此方法的别名。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-string-left-right-trim)

#### 动机

ES5 标准化了 `String.prototype.trim`。
所有主要搜索引擎也实现了相应的 `trimLeft` 和 `trimRight` 功能，但是无任何标准规范。
为了与 `padStart/padEnd` 保持一致，所以提议使用 `trimStart` 和 `trimEnd`。
考虑到不同浏览器的兼容性问题，`trimLeft/trimRight` 将作为 `trimStart/trimEnd` 的别名使用。

```javascript
String.prototype.trimLeft === String.prototype.trimStart;
// true
String.prototype.trimRight === String.prototype.trimEnd;
// true
```

#### 语法

```javascript
const greeting = '   Hello world!   ';
console.log(greeting.trimStart());
// "Hello world!   ";
console.log(greeting.trimEnd());
// "   Hello world!";
```

### 4、Optional Catch Binding

#### 概述

> 该提案对 ECMAScript 进行了语法更改，允许在不使用 catch 绑定的情况下省略 catch 绑定。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-optional-catch-binding)

#### 动机

```javascript
try {
  // try to use a web feature which may not be implemented
} catch (unused) {
  // fall back to a less desirable web feature with broader support
}
```

```javascript
let isTheFeatureImplemented = false;
try {
  // stress the required bits of the web API
  isTheFeatureImplemented = true;
} catch (unused) {}
```

普遍认为，声明或写入从未读取的变量会引发程序错误。

#### 语法

```javascript
try {
  // ...
} catch {
  // ...
}
```

### 5、Symbol.prototype.description

#### 概述

> `Symbol` 对象可以通过一个可选的描述创建，可用于调试，但不能用于访问 `symbol` 本身。
>
> `Symbol.prototype.description` 属性可以用于读取该描述。与 `Symbol.prototype.toString()` 不同的是它不会包含 `"Symbol()"` 的字符串。
> 
> `description` 是一个只读属性，它会返回 `Symbol` 对象的可选描述的字符串。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-Symbol-description)

#### 动机

直接暴露 `symbol` 的 `description` 内部插槽，取代通过 `symbol.prototype.toString` 间接暴露。

#### 语法

```javascript
Symbol('myDescription').description;

Symbol.iterator.description;

Symbol.for('foo').description;
```

**例子：**

```javascript
Symbol('desc').toString();  // "Symbol(desc)"
Symbol('desc').description; // "desc"
Symbol('').description;     // ""
Symbol().description;       // undefined

// well-known symbols
Symbol.iterator.toString();  // "Symbol(Symbol.iterator)"
Symbol.iterator.description; // "Symbol.iterator"

// global symbols
Symbol.for('foo').toString();  // "Symbol(foo)"
Symbol.for('foo').description; // "foo"
```

## ES11（ES2020）

### 1、String.prototype.matchAll

#### 概述

> `matchAll()` 方法返回一个包含所有匹配正则表达式的结果及分组捕获组的迭代器。
> 
> [TC39 直通车](https://github.com/tc39/proposal-string-matchall)

#### 动机

之前，如果我们要获取所有匹配项信息，只能通过循环调用 `regexp.exec()`（ regexp 需使用 `/g` 标志）：

```javascript
const regexp = RegExp('foo[a-z]*', 'g');
const str = 'table football, foosball';
let match;

while ((match = regexp.exec(str)) !== null) {
  console.log(`Found ${match[0]} start = ${match.index} end = ${regexp.lastIndex}.`);
  // expected output: "Found football start=6 end=14."
  // expected output: "Found foosball start=16 end=24."
}
```

如果使用 `matchAll`，就可以不必使用 `while` 循环加 `exec` 方式（且正则表达式需使用 `/g` 标志）。使用 `matchAll` 会得到一个迭代器的返回值，配合 for...of、array spread 或者 Array.from() 可以更方便实现功能。

#### 语法

> str.matchAll(regexp)
> 
> - `regexp`：正则表达式对象。如果所传参数不是一个正则表达式对象，则会隐式地使用 `new RegExp(obj)` 将其转换为一个 `RegExp` 

`RegExp` 必须是设置了全局模式 `/g` 的形式，否则会抛出异常 `TypeError`。

**例子：**

```javascript
const regexp = RegExp('foo[a-z]*', 'g');
const str = 'table football, foosball';
const matches = str.matchAll(regexp);

for (const match of matches) {
  console.log(`Found ${match[0]} start = ${match.index} end = ${match.index + match[0].length}.`);
}
// expected output: "Found football start=6 end=14."
// expected output: "Found foosball start=16 end=24."

// matches iterator is exhausted after the for..of iteration
// Call matchAll again to create a new iterator
Array.from(str.matchAll(regexp), m => m[0]);
// Array [ "football", "foosball" ]
```

#### 注意事项

1、如果没有 `/g` 标志，`matchAll` 会抛出异常。

```javascript
const regexp = RegExp('[a-c]', '');
const str = 'abc';
Array.from(str.matchAll(regexp), m => m[0]);
// TypeError: String.prototype.matchAll called with a non-global RegExp argument
```

2、`matchAll` 内部做了一个 `regexp` 的复制，所以不像 `regexp.exec`，`lastIndex` 在字符串扫描时不会改变。

```javascript
const regexp = RegExp('[a-c]', 'g');
regexp.lastIndex = 1;
const str = 'abc';
Array.from(str.matchAll(regexp), m => `${regexp.lastIndex} ${m[0]}`);
// Array [ "1 b", "1 c" ]
```

3、`matchAll` 的另外一个亮点是更好地获取捕获组。因为当使用 `match()` 和 `/g` 标志方式获取匹配信息时，捕获组会被忽略：

```javascript
const regexp = /t(e)(st(\d?))/g;
const str = 'test1test2';

str.match(regexp);
// Array ['test1', 'test2']
```

使用 `matchAll` 可以通过如下方式获取分组捕获：

```javascript
let array = [...str.matchAll(regexp)];

array[0];
// ['test1', 'e', 'st1', '1', index: 0, input: 'test1test2', length: 4]
array[1];
// ['test2', 'e', 'st2', '2', index: 5, input: 'test1test2', length: 4]
```

### 2、Dynamic Import

#### 概述

> 动态导入模块
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-dynamic-import)

#### 动机

标准用法的 `import` 导入的模块是静态的，会使所有被导入的模块，在加载时就被编译（无法做到按需编译，降低首页加载速度）。有些场景中，你可能希望根据条件导入模块或者按需导入模块，这时你可以使用动态导入代替静态导入。下面的是你可能会需要动态导入的场景：

- 当静态导入的模块很明显的降低了代码的加载速度且被使用的可能性很低，或者并不需要马上使用它。
- 当静态导入的模块很明显的占用了大量系统内存且被使用的可能性很低。
- 当被导入的模块，在加载时并不存在，需要异步获取。
- 当导入模块的说明符，需要动态构建。（静态导入只能使用静态说明符）。
- 当被导入的模块有副作用（这里说的副作用，可以理解为模块中会直接运行的代码），这些副作用只有在触发了某些条件才被需要时。（原则上来说，模块不能有副作用，但是很多时候，你无法控制你所依赖的模块的内容）。

#### 语法

> import()

关键字 `import` 可以像调用函数一样来动态的导入模块。以这种方式调用，将返回一个 `promise`。

**例子：**

```javascript
import('/modules/my-module.js')
  .then((module) => {
    // Do something with the module.
  });
```

也支持 `await` 关键字

```javascript
const module = await import('/modules/my-module.js');
```

模板字符串形式

```javascript
const main = document.querySelector("main");
  for (const link of document.querySelectorAll("nav > a")) {
    link.addEventListener("click", e => {
      e.preventDefault();
      import(`./section-modules/${link.dataset.entryModule}.js`)
        .then(module => {
          module.loadPageInto(main);
        })
        .catch(err => {
          main.textContent = err.message;
        });
    });
  }
```

#### 注意事项

请不要滥用动态导入（只有在必要情况下采用）。
静态框架能更好地初始化依赖，而且更有利于静态分析工具和 `tree shaking` 发挥作用。

与静态 `import` 差异：

- `import()` 可以从脚本中使用，而不仅仅是从模块中使用。
- 如果 `import()` 在模块中使用，它可以出现在任何级别的任何地方，并且不会被提升。
- `import()` 接受任意字符串（此处显示了运行时确定的模板字符串），而不仅仅是静态字符串文字。
- 模块中存在 `import()` 并不会建立依赖关系，在对包含的模块求值之前，必须先获取并求值依赖关系。
- `import()` 不建立可以静态分析的依赖关系（但是，在诸如 `import('./foo.js')` 这样的简单情况下，实现可能仍然能够执行推测性抓取）。

### 3、import.meta

#### 概述

> `import.meta` 是一个给 `JavaScript` 模块暴露特定上下文的元数据属性的对象。它包含了这个模块的信息，比如说这个模块的 `URL`。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-import-meta)

#### 动机

通常情况下，主机环境可以提供有用的模块特定信息，以便在模块内进行代码评估。以下是几个例子。

**模块的 URL 或 文件名**

这是在 Node.js 的 CommonJS 模块系统中通过作用域内 `__filename` 变量（及其对应的 `__dirname` ）提供的。这允许通过代码轻松解析相对于模块文件的资源，例如：

```javascript
const fs = require("fs");
const path = require("path");
const bytes = fs.readFileSync(path.resolve(__dirname, "data.bin"));
```

如果没有可用的 `_dirname`，像 `fs.readFileSync('data.bin')` 这样的代码将解析相对于当前工作目录的 `data.bin`。这对于库作者来说通常是没有用的，因为他们将自己的资源与模块捆绑在一起，这些模块可以位于与 CWD 相关的任何位置。

浏览器中也存在相似的行为，将文件名替换为 URL。

**scrpit 标签引入资源**

```javascript
<script data-option="value" src="library.js"></script>

// 获取 data-option 的值如下
const theOption = document.currentScript.dataset.option;
```

为此使用（有效的）全局变量而不是词法范围的值的机制是有问题的，因为这意味着该值仅设置在顶级，并且必须保存在那里才能用于任何异步代码。

**mainModule**

在 Node.js 中，通常的做法是通过以下代码来确定您是程序的 `main` 模块还是 `entry` 模块：

```javascript
if (module === process.mainModule) {
  // run tests for this library, or provide a CLI interface, or similar
}
```

也就是说，它允许单个文件在导入时用作库，或者在直接运行时用作副作用程序。

请注意，此特定公式如何依赖于将主机提供的作用域值模块与主机提供的全局值 `process.mainModule` 进行比较。

**其他可设想的情况**

- 其他 `Node.js` 方法，如 `module.children` 或 `require.resolve()`
- 关于模块所属 `package` 的信息，或者通过 `Node.js` 的 `package.json` 或 `web package`
- 指向嵌入在 `HTML` 模块中的 `JavaScript` 模块的包含 `DocumentFragment` 的指针

#### 语法

> import.meta

`import.meta` 对象由一个关键字 `import`，一个`点符号`和一个 `meta` 属性名组成。通常情况下 `import.` 是作为一个属性访问的上下文，但是在这里 `import` 不是一个真正的对象。

`import.meta` 对象带有一个 `null` 的原型对象。这个对象可以扩展，并且它的属性都是可写，可配置和可枚举的。

**例子：**

以下代码使用了我们添加到 `import.meta` 中的两个属性：

```javascript
(async () => {
  const response = await fetch(new URL("../hamsters.jpg", import.meta.url));
  const blob = await response.blob();

  const size = import.meta.scriptElement.dataset.size || 300;

  const image = new Image();
  image.src = URL.createObjectURL(blob);
  image.width = image.height = size;

  document.body.appendChild(image);
})();
```

当这个模块被加载时，无论它的位置如何，它都会加载 `hamsters.jpg` 文件，并显示图像。可以使用用于导入它的脚本元素来配置图像的大小，例如：

```javascript
<script type="module" src="path/to/hamster-displayer.mjs" data-size="500"></script>
```

### 4、BigInt

#### 概述

> 在 JavaScript 中，`BigInt` 是一种数字类型的数据，它可以表示任意精度格式的整数。（新的基本数据类型）
>
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-bigint)

#### 动机

在 JavaScript 中，当 `number` 大于 `Number.MAX_SAFE_INTEGER` 时，就会发生精度丢失。

```javascript
const x = Number.MAX_SAFE_INTEGER;
// 9007199254740991, this is 1 less than 2^53

const y = x + 1;
// 9007199254740992, ok, checks out

const z = x + 2;
// 9007199254740992, wait, that’s the same as above!
```

`BigInt` 的提案正是要解决这个问题，`BigInt` 使 `Number.MAX_SAFE_INTEGER` 不再是 JavaScript 的限制。它可以表示任意精度的整数。

#### 语法

> 通过 `BigInt` 方法，或是在一个数值后添加 `n` 后缀，来将一个 Number 转换为 BigInt 类型。

**例子：**

```javascript
const theBiggestInt = 9007199254740991n;
const alsoHuge = BigInt(9007199254740991);
// 9007199254740991n
const hugeButString = BigInt('9007199254740991');
// 9007199254740991n
```

```javascript
Number.MAX_SAFE_INTEGER;
// 9007199254740991
Number.MAX_SAFE_INTEGER + 10 - 10;
// 9007199254740990   （精度丢失）
BigInt(Number.MAX_SAFE_INTEGER) + 10n - 10n;
// 9007199254740991n  （计算结果为 bigint 类型）

typeof 9007199254740991n === "bigint";
// true
```

你可以使用 `+`、`*`、`-`、`**`、`%` 运算符，就像在 Number 中使用一样。

```javascript
const previousMaxSafe = BigInt(Number.MAX_SAFE_INTEGER);
// 9007199254740991

const maxPlusOne = previousMaxSafe + 1n;
// 9007199254740992n
 
const theFuture = previousMaxSafe + 2n;
// 9007199254740993n, this works now!

const multi = previousMaxSafe * 2n;
// 18014398509481982n

const subtr = multi – 10n;
// 18014398509481972n

const mod = multi % 10n;
// 2n

const bigN = 2n ** 54n;
// 18014398509481984n

bigN * -1n
// –18014398509481984n
```

#### 注意事项

1、`/` 除法运算符也可以使用，但是会向下取整。

因为 `BigInt` 不是 `BigDecimal`。

```javascript
const expected = 4n / 2n; // 2n
const rounded = 5n / 2n;  // 2n, not 2.5n
```

2、`BigInt` 并不严格等于 `Number`。

```javascript
2n === 2; // false
2n == 2;  // true
```

3、比较运算符可以正常使用。

```javascript
1n < 2;  // true 
2n > 1;  // true 
2 > 2;   // false 
2n > 2;  // false 
2n >= 2; // true
```

4、数组中混合 `BigInt` 和 `Number` 可以正常排序。

```javascript
const mixed = [4n, 6, -12n, 10, 4, 0, 0n];
// [4n, 6, -12n, 10, 4, 0, 0n]

mixed.sort();
// [-12n, 0, 0n, 10, 4n, 4, 6]
```

5、`BigInt` 不可以与 `Number` 混合运算。

```javascript
1n + 2;
// TypeError: Cannot mix BigInt and other types, use explicit conversions

1n * 2;
// TypeError: Cannot mix BigInt and other types, use explicit conversions
```

### 5、Promise.allSettled

#### 概述

> `Promise.allSettled()` 方法返回一个在所有给定的 `promise` 都已经 `fulfilled` 或 `rejected` 后的 `promise`，并带有一个对象数组，每个对象表示对应的 `promise` 结果。
>
> 当您有多个彼此不依赖的异步任务成功完成时，或者您总是想知道每个 `promise` 的结果时，通常使用它。
>
> 相比之下，`Promise.all()` 更适合彼此相互依赖或者在其中任何一个 `reject` 时立即结束。
>
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-promise-allSettled)

#### 动机

| api | 描述 |  |
| ---- | ---- | ---- |
| Promise.allSettled | 不会短路 | 本提案 🆕 |
| Promise.all | 当存在输入值 rejected 时短路 | 在 ES2015 中添加 ✅ |
| Promise.race | 当存在输入值 稳定 时短路 | 在 ES2015 中添加 ✅ |
| Promise.any | 当存在输入值 fulfilled 时短路 | 在 ES2021 中添加 ✅ |

可以看出，`Promise.all`、`Promise.race`、`Promise.any` 都存在短路行为。

`Promise.allSettled` 不存在短路行为，它确保每个 `promise` 都可以执行完毕，无论状态是 `fulfilled` 还是  `rejected`。

#### 语法

> Promise.allSettled(iterable)
> 
> - `iterable`：一个可迭代的对象，例如 `Array`，其中每个成员都是 `Promise`。

一旦所指定的 `promises` 集合中每一个 `promise` 已经完成，无论是成功的达成或被拒绝，未决议的 `promise` 将被异步完成。那时，所返回的 `promise` 的处理器将传入一个数组作为输入，该数组包含原始 `promises` 集中每个 `promise` 的结果。

对于每个结果对象，都有一个 `status` 字符串。如果它的值为 `fulfilled`，则结果对象上存在一个 `value`。如果值为 `rejected`，则存在一个 `reason`。`value`（或 `reason`）反映了每个 `promise` 决议（或拒绝）的值。

**例子：**

```javascript
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
const promises = [promise1, promise2];

Promise.allSettled(promises)
  .then((results) => results.forEach((result) => console.log(result.status)));

// expected output:
// "fulfilled"
// "rejected"
```

```javascript
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise((resolve, reject) => reject('我是失败的Promise_1'));
const promise4 = new Promise((resolve, reject) => reject('我是失败的Promise_2'));
const promiseList = [promise1, promise2, promise3, promise4];
Promise.allSettled(promiseList)
.then(values => {
  console.log(values);
});
```

### 6、globalThis

#### 概述

> 全局属性 `globalThis` 包含全局的 `this` 值，类似于全局对象（global object）。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-global)

#### 动机

编写访问全局对象的可移植 ECMAScript 代码是很困难的。在 `web` 端，可以是 `window` 或者 `self` 或者 `this` 或者 `frames`。在 `node.js` 中，是 `global` 或者 `this`。

其中，只有 `this` 在 V8 的 `d8` 或者 JavaScriptCore 的 `jsc` 这样的 `shell` 中可用。
在 `sloppy` 模式下的独立函数调用中，`this` 也可以工作，但是在模块中或在函数内的严格模式下它是 `undefined`。

在这样的上下文中，全局对象仍然可以通过 `Function('return this')()` 访问，但在某些 CSP 设置中，例如在 Chrome 应用程序中是不可访问的。

下面是一些从外部获取全局对象的代码，作为单个参数传递给 IIFE，它在大多数情况下都有效，但实际上不适用于 `d8` 当在模块中或在函数内的严格模式下（可以使用函数技巧修复）：

```javascript
function foo() {
  // If we're in a browser, the global namespace is named 'window'. If we're
  // in node, it's named 'global'. If we're in a shell, 'this' might work.
  (typeof window !== "undefined"
    ? window
    : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
      ? global
      : this);
}
```

此外，由于 CSP 问题，es6-shim 必须从 `Function('return this')()` 转换，
当前处理 `browsers`、`node`、`web workers`、`frames` 方式如下：

```javascript
const getGlobal = function () {
  // the only reliable means to get the global object is
  // `Function('return this')()`
  // However, this causes CSP violations in Chrome apps.
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```

#### 语法

在浏览器中， `window`、`globalThis`、`this` 三者全等：

```javascript
window === this;       // true
window === globalThis; // true
this === globalThis;   // true
```

在 node.js 中，`global`、`globalThis`、`this` 三者全等：

```javascript
> global === this;       // true
> global === globalThis; // true
> this === globalThis;   // true
```

### 7、Nullish Coalescing

#### 概述

> 空值合并操作符（`??`）是一个逻辑操作符，当左侧的操作数为 `null` 或者 `undefined` 时，返回其右侧操作数，否则返回左侧操作数。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-nullish-coalescing)

#### 动机

在执行属性访问时，如果属性访问的结果为 `null` 或 `undefined`，则通常需要提供默认值。目前，在 JavaScript 中表达此意图的一种典型方式是使用 `||` 运算符。

```javascript
const response = {
  settings: {
    nullValue: null,
    height: 400,
    animationDuration: 0,
    headerText: '',
    showSplashScreen: false,
  },
};
const undefinedValue = response.settings.undefinedValue || 'some other default';
// result: 'some other default'
const nullValue = response.settings.nullValue || 'some other default';
// result: 'some other default'
```

这对于 `null` 和 `undefined` 的值常见情况非常有效，但有许多 `falsy` 值可能会产生令人惊讶的结果：

```javascript
const headerText = response.settings.headerText || 'Hello, world!';
// Potentially unintended. '' is falsy, result: 'Hello, world!'
const animationDuration = response.settings.animationDuration || 300;
// Potentially unintended. 0 is falsy, result: 300
const showSplashScreen = response.settings.showSplashScreen || true;
// Potentially unintended. false is falsy, result: true
```

#### 语法

> leftExpr ?? rightExpr

**例子：**

```javascript
let user = {
  u1: 0,
  u2: false,
  u3: null,
  u4: undefined,
  u5: '',
}
let u2 = user.u2 ?? '用户2'; // false
let u3 = user.u3 ?? '用户3'; // 用户3
let u4 = user.u4 ?? '用户4'; // 用户4
let u5 = user.u5 ?? '用户5'; // ''
```

### 8、Optional Chaining

#### 概述

> 可选链操作符（`?.`）允许读取位于连接对象链深处的属性的值，而不必明确验证链中的每个引用是否有效。`?.` 操作符的功能类似于 `.` 链式操作符，不同之处在于，在引用（`null` 或者 `undefined`）的情况下不会引起错误，该表达式短路返回值是 `undefined`。与函数调用一起使用时，如果给定的函数不存在，则返回 `undefined`。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-optional-chaining)

#### 动机

在寻找对象结构深处的属性值时，通常必须检查中间节点是否存在：

```javascript
const street = user.address && user.address.street;
```

此外，许多 `API` 返回对象或 `null/undefined`，并且可能希望仅在结果不为 `null` 时从结果中提取属性：

```javascript
const fooInput = myForm.querySelector('input[name=foo]');
const fooValue = fooInput ? fooInput.value : undefined;
```

#### 语法

> obj?.prop
> 
> obj?.[expr]
> 
> arr?.[index]
> 
> func?.(args)

**例子：**

使用可选链访问对象属性：

```javascript
let user = {};
let u1 = user.childer.name;
// TypeError: Cannot read property 'name' of undefined
let u1 = user?.childer?.name;
// undefined
```

使用可选链进行函数调用

```javascript
function doSomething(onContent, onError) {
  try {
   // ... do something with the data
  }
  catch (err) {
    onError?.(err.message); // 如果onError是undefined也不会有异常
  }
}
```

当使用方括号与属性名的形式来访问属性时，你也可以使用可选链操作符：

```javascript
const nestedProp = obj?.['prop' + 'Name'];
```

可选链访问数组元素：

```javascript
let arrayItem = arr?.[42];
```

## ES12（ES2021）

### 1、String.prototype.replaceAll

#### 概述

> `replaceAll()` 方法返回一个新字符串，新字符串所有满足 `pattern` 的部分都已被 `replacement` 替换。`pattern` 可以是一个字符串或一个 `RegExp`， `replacement` 可以是一个字符串或一个在每次匹配被调用的函数。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-string-replaceall)

#### 动机

目前没有办法在不使用全局正则表达式的情况下替换字符串中子字符串的所有实例。 `String.prototype.replace` 仅在与字符串参数一起使用时替换掉第一次出现的子字符串。

目前实现这一目标的最常见方法是使用全局正则表达式：

```javascript
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.replace(/\+/g, ' ');
```

这种方法的缺点是需要对特殊的 `RegExp` 字符进行转义——注意转义的 `+`。

另一种解决方案是结合 `String.split` 和 `Array.join`：

```javascript
const queryString = 'q=query+string+parameters';
const withSpaces = queryString.split('+').join(' ');
```

这种方法避免了任何转义，但带来了将字符串拆分为数组，然后再将数组各项重新粘合在一起的开销。

#### 语法

> String.prototype.replaceAll(searchValue, replaceValue);

返回一个全新的字符串，所有符合匹配规则的字符都将被替换掉。

除了以下两种情况外，在其他所有情况下 `String.prototype.replaceAll` 行为都与 `String.prototype.replace` 相同：

- 如果 `searchValue` 是一个字符串，`String.prototype.replace` 只替换一次出现的 `searchValue`，而 `String.prototype.replaceAll` 替换所有出现的 `searchValue`。
- 如果 `searchValue` 是非全局正则表达式，则 `String.prototype.replace` 替换单个匹配项，而 `String.prototype.replaceAll` 抛出异常。这样做是为了避免缺少全局标志（这意味着“不要全部替换”）和被调用方法的名称（强烈建议“全部替换”）之间的固有混淆。

**例子：**

```javascript
const queryString = 'q=query+string+parameters'; 
const withSpaces = queryString.replaceAll('+', ' ');
```

### 2、Promise.any

#### 概述

> `Promise.any()` 接收一个 `Promise` 可迭代对象，只要其中的一个 `promise` 成功，就返回那个已经成功的 `promise`。如果可迭代对象中没有一个 `promise` 成功（即所有的 `promises` 都失败/拒绝），就返回一个失败的 `promise`。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-promise-any)

#### 语法

> Promise.any(iterable)
> - iterable：一个可迭代的对象，例如 Array。

**例子：**

```javascript
const promise1 = new Promise((resolve, reject) => reject('失败的Promise1'));
const promise2 = new Promise((resolve, reject) => reject('失败的Promise2'));
const promiseList = [promise1, promise2];
Promise.any(promiseList)
.then(values=>{
  console.log(values);
})
.catch(error=>{
  console.log(error);
});
```

在上面的示例中，`error` 是 `AggregateError`，它是一个新的 `error` 子类，将各个错误组合在一起。每个 `AggregateError` 实例都包含一个指向异常数组的指针。

### 3、Logical Assignment Operators

#### 概述

> 逻辑赋值运算符
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-logical-assignment)

#### 语法

> `||=`
> 
> `&&=`
> 
> `??=`

**例子：**

```javascript
a ||= b;
//等价于
a = a || (a = b);

a &&= b;
//等价于
a = a && (a = b);

a ??= b;
//等价于
a = a ?? (a = b);
```

### 4、Numeric Separators

#### 概述

> 数字分隔符
>
> 它是将其早期草案与 `Christophe Porteneuve` 的提案 `numeric underscores` 合并的结果，以扩展现有的 `NumericLiteral` 以允许数字之间的分隔符。
> 
> 该提案目前处于 Stage 4 阶段。
> 
> [TC39 直通车](https://github.com/tc39/proposal-numeric-separator)

#### 动机

此功能使开发人员能够通过在数字之间创建视觉分隔来使他们的数字文字更具可读性。

#### 语法

> 使用下划线 `_`（U+005F）在数字间进行分隔。

**例子：**

十进制

```javascript
const price1 = 1_000_000_000;
const price2 = 1000000000;
price1 === price2; // true

const float1 = 0.000_001;
const float2 = 0.000001;
float1 === float2; // true;
```

二进制

```javascript
const nibbles1 = 0b1010_0001_1000_0101;
const nibbles2 = 0b1010000110000101;
nibbles1 === nibbles2; // true
```

十六进制

```javascript
const message1 = 0xA1_B1_C1;
const message2 = 0xA1B1C1;
message1 === message2; // true
```

BigInt

```javascript
const big1 = 1_000_000_000n;
const big2 = 1000000000n;
big1 === big2; // true
```
