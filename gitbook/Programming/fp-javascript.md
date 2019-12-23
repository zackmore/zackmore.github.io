# Professor Frisby's Mostly Adequate Guide to Functional Programming

https://github.com/MostlyAdequate/mostly-adequate-guide

## 2) 一等公民的函数

> 当我们说函数是“一等公民”的时候，我们实际上说的是它们和其他对象都一样...函数真没什么特殊的，你可以像对待任何其他数据类型一样对待它们——把它们存在数组里，当作参数传递，赋值给变量...等等

```javascript
// A bad example...
const hi = name => `Hi ${name}`;
const greeting = name => hi(name);

// Could change to...
const greeting = hi;
greeting("times");
```

```javascript
// A worse example...
const getServerStuff = callback => ajaxCall(json => callback(json));

// Could change to...
const getServerStuff = callback => ajaxCall(callback);

// Could change to...
const getServerStuff = ajaxCall;
```

```javascript
// A worst example...
const BlogController = {
  index(posts) { return Views.index(posts); },
  show(post) { return Views.show(post); },
  create(attrs) { return DB.create(attrs); },
  update(post, attrs) { return DB.update(post, attrs); },
  destroy(post) { return DB.destroy(post); },
};

// Could change to...
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: DB.create,
  update: DB.update,
  destroy: DB.destroy,
};

// Or remove it, it just binds Views and DB
```

> 用一个函数把另一个函数包起来，目的仅仅是延迟执行，真的是非常糟糕的编程习惯。（跟可维护性密切相关）

**如无必要，勿增实体**

## 3) 纯函数的好处

纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且没有任何可观察的副作用。

不纯的函数运行结果取决于 system state，引入了外部环境，增加了 cognitive load。这种依赖状态是影响系统复杂度的罪魁祸首。输入之外的因素能够左右结果，不仅让这个函数变得不纯，而且导致每次我们思考整个软件的时候都痛苦不堪。

纯函数能够自给自足，也可以让外部因素变成 immutable 状态，这样就能保留纯粹性。

作用可以理解为一切除结果计算之外发生的事情，作用本身并没有什么坏处，副作用的关键部分在于“副”，是滋生 bug 的温床。

只要是跟函数外部环境发生的交互就都是副作用——这一点可能会让你怀疑无副作用编程的可行性。函数式编程的哲学就是**假定副作用是造成不正当行为的主要原因**。

这并不是说，要禁止使用一切副作用，而是说，要让它们在可控的范围内发生。

副作用让一个函数变得不纯是有道理的：从定义上来说，纯函数必须要能够根据相同的输入返回相同的输出；如果函数需要跟外部事物打交道，那么就无法保证这一点了。

---

纯函数优点:

- 可缓存性 Cacheable
  - 固定输入对应固定输出，所以天然可缓存
- 可移植性/自文档化 Portable/Self-Documenting
  - 自给自足
  - 通过强迫注入依赖，或者把依赖当作参数来传递，达成与环境无关
- 可测试性 Testable
  - 固定输入对应固定输出
  - Quickcheck
- 合理性 Reasonable
  - 固定输入对应固定输出
  - 引用透明性，如果一段代码可以替换成它执行所得的结果，而且是在不改变整个程序行为的前提下替换的，那么我们就说这段代码是引用透明的
  - 等式推导
- 并行代码
  - 纯函数不需要访问共享内存，所以也不会因副作用进入竞争态 race condition

---

> I think the lack of reusability comes in object-oriented languages, not functional languages. Because the problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana and the entire jungle.
> If you have referentially transparent code, if you have pure functions — all the data comes in its input arguments and everything goes out and leave no state behind — it’s incredibly reusable.
> -- Joe Armstrong, creator of Erlang, on software reusability.

## 4) Curry

**只传递函数一部分来调用它，让它返回一个函数去处理剩下的参数**

`HOF`: Higher Order Functions 高阶函数，参数或返回值为函数的函数

策略性的把要操作的数据放到最后一个参数，有利于 curry。

通过简单地传递几个参数，就能动态创建实用的新函数；而且还能带来一个额外好处，那就是保留了数学的函数定义，尽管参数不止一个。

```javascript
const add = x => y => x + y;

const add1 = add(1);
const add10 = add(10);

add1(2);
// 3

add10(5);
// 15
```

---

`ramda.curry` return a curried equivalent of provided function. The curried function has two unusual capabilities. First, its arguments needn't be provided one at a time. If `f` is a ternary function and `g` is `R.curry(f)`, the following are equivalent:

```
- g(1)(2)(3)
- g(1)(2, 3)
- g(1, 2)(3)
- g(1, 2, 3)
```

Secondly, the special placeholder value `R.__` may be used to specify "gaps", allowing partial application of any combination of arguments, regardless of their positions. if `g` is as above and `_` is `R.__`, th folloing are equivalent:

```
- g(1, 2, 3)
- g(_, 2, 3)(1)
- g(_, _, 3)(1)(2)
- g(_, _, 3)(1, 2)
- g(_, 2)(1)(3)
- g(_, 2)(1, 3)
- g(_, 2)(_, 3)(1)
```

## 5) Compose

```javascript
const toUpperCase = x => x.toUpperCase();
const exclaim = x => `${x}!`;

// With Compose
const shout = compose(exclaim, toUpperCase);

// No Compose
const shoutNormal = x => exclaim(toUpperCase(x));

shout('hello world');
shoutNormal('hello world');
// => "HELLO WORLD!"
```

让代码从右向左执行，而不是由内而外执行。从右向左更加能够反映数学上的含义，因为组合的概念直接来源于数学。

**结合律**

```javascript
const f1 = compose(f, compose(g, h));
const f2 = compose(compose(f, g), h);
const f3 = compose(f, g, h);

f1 == f2 && f1 == f3 && f2 == f3;
// => true
```

如何为 `compose` 的调用分组不重要，因为结果一样。对于多个组合，既然符合结合律，我们就可以只写一个组合，想传给它多少个函数就传多少，由它自己决定分组。

如何组合没有标准答案，通常最佳实践是让组合可重用。

**pointfree**

函数无须提及将要操作的数据是什么样的。

```javascript
// non-pointfree
const snakeCase = word => word.toLowerCase().replace(/\s+/ig, '_');

// pointfree
const replace = curry((reg, replacement, str) => str.replace(reg, replacement));
const toLowerCase = curry(str => str.toLowerCase());
const snakeCasePF = compose(replace(/\s+/ig, '_'), toLowerCase);
```

**debug**

组合的一个常见错误，在没有局部调用之前，就组合类似 `map` 这样接受两个参数的函数。

```javascript
// 错误做法，传给了 angry 一个数组，不知道最后传给 map 的是什么东西
const latin = compose(map, angry, reverse);
latin(['frog', 'eyes']);
// error

// 正确做法：每个函数都接受一个实际参数
const latin = compose(map(angry), reverse);
latin(['frog', 'eyes']);
// ["EYES!", "FROG!"]
```

如果在 debug 组合的时候遇到了困难，可以用下面这个不纯的 `trace` 函数来追踪代码执行情况。

```javascript
const trace = curry((tag, x) => {
  console.log(tag, x);
  return x;
});

// 用在上例里
const latin = compose(
  map,
  trace('after angry'),
  angry,
  trace('after reverse'),
  reverse,
);
```

## 6) 示例应用

**声明式代码**

不指示计算机如何工作，而是指出我们明确希望得到的结果。这意味着声明式要写表达式，而不是一步一步的指示。

```javascript
// 命令式
var makes = [];
for (i = 0; i < cars.length; i++) {
  makes.push(cars[i].make);
}

// 声明式
var makes = cars.map(car => car.make);
```

命令式的循环要求你必须先实例化一个数组，而且执行完这个实例化语句之后，解释器才继续执行后面的代码。然后再直接迭代 cars 列表，手动增加计数器，把各种零零散散的东西都展示出来...实在是直白得有些露骨。

使用 map 的版本是一个表达式，它对执行顺序没有要求。而且，map 函数如何进行迭代，返回的数组如何收集，都有很大的自由度。它指明的是**做什么**，不是**怎么做**。因此，它是正儿八经的声明式代码。

另一个例子：

```javascript
// 命令式
const authenticate = form => {
  const user = toUser(form);
  return logIn(user);
};
// 硬编码了一步接一步的执行方式

// 声明式
const authenticate = compose(
  logIn,
  toUser,
);
// 用户验证是 toUser 和 logIn 两个行为的组合
```

**TODO: how to compose Promise?**

## 7) Hindley-Milner 类型签名

**一些例子：**

```javascript
// strLength :: String -> Number
const strLength = s => s.length;

// join :: String -> [String] -> String
const join = curry((what, xs) => xs.join(what));

// match :: Regex -> String -> [String]
const match = curry((reg, s) => s.match(reg));

// replace :: Regex -> String -> String -> String
const replace = curry((reg, sub, s) => s.replace(reg, sub));

// onHoliday :: String -> [String]
const onHoliday = match(/holiday/ig);

// id :: a -> a
const id = x => x;

// map :: (a -> b) -> [a] -> [b]
const map = curry((f, xs) => xs.map(f));

// head :: [a] => a
const head = xs => xs[0];

// filter :: (a -> Bool) -> [a] -> [a]
const filter = curry((f, xs) => xs.filter(f));

// reduce :: (b -> a -> b) -> b -> [a] -> b
const reduce = curry((f, x, xs) => xs.reduce(f, x));
```

**缩小可能性范围** (narrowing of possibility)

`_parametricity_` 函数将会以一种统一的行为作用于所有的类型。

```javascript
// head :: [a] -> a
```

注意看 head ，可以看到它接受 [a] 返回 a。我们除了知道参数是个数组，其他的一概不知；所以函数的功能就只限于操作这个数组上。在它对 a 一无所知的情况下，它可能对 a 做什么操作呢？换句话说，a 告诉我们它不是一个特定的类型，这意味着它可以是任意类型；那么我们的函数对每一个可能的类型的操作都必须保持统一。这就是 `_parametricity_` 的含义。要让我们来猜测 head 的实现的话，唯一合理的推断就是它返回数组的第一个，或者最后一个，或者某个随机的元素；当然，head 这个命名应该能给我们一些线索。

```javascript
// reverse :: [a] -> [a]
```

仅从类型签名来看，reverse 可能的目的是什么？再次强调，它不能对 a 做任何特定的事情。它不能把 a 变成另一个类型，或者引入一个 b；这都是不可能的。那它可以排序么？答案是不能，没有足够的信息让它去为每一个可能的类型排序。它能重新排列么？可以的，我觉得它可以，但它必须以一种可预料的方式达成目标。另外，它也有可能删除或者重复某一个元素。重点是，不管在哪种情况下，类型 a 的多态性（polymorphism）都会大幅缩小 reverse 函数可能的行为的范围。

**自由定理** (free theorems)

```javascript
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) == compose(filter(p), map(f));
```

**类型约束** (type constraints)

签名可以把类型约束为一个特定的接口 interface

```javascript
// sort :: Ord a => [a] -> [a]
```

胖箭头左边表明 a 一定是个 Ord 对象。Ord 到底是什么？它是从哪来的？在一门强类型语言中，它可能就是一个自定义的接口，能够让不同的值排序。通过这种方式，我们不仅能够获取关于 a 的更多信息，了解 sort 函数具体要干什么，而且还能限制函数的作用范围。我们把这种接口声明叫做类型约束（type constraints）。

```javascript
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

这个例子中有两个约束：Eq 和 Show。它们保证了我们可以检查不同的 a 是否相等，并在有不相等的情况下打印出其中的差异。

## 8) Tupperware

**Container _(Identity / id)_**

函数式程序即通过管道把数据在一系列纯函数间传递的程序，这些程序就是声明式的行为规范。但如何处理控制流 `_control flow_`、异常处理 `_error handling_`、异步操作 `_asynchronous actions_` 和状态 `_state_` 呢？还有更棘手的作用 `_effects_`？

首先我们需要一个容器，一个可以存放任意类型值的容器。

```javascript
const Container = function(x) {
  this.__value = x;
}

Container.of = x => new Container(x);
```

- Container 是只有一个名为 __value 属性的对象
- __value 不能是某个特定的类型
- 数据一旦存放到 Container，就会一直待在那儿。我们可以用 .__value 获取数据，但这样有悖于初衷

---

**Functor _(mappable container)_**

然后我们需要一种方式来操作容器。

```javascript
// (a -> b) -> Container a -> Container b
Container.prototype.map = function(x) {
  return Container.of(f(this.__value));
}
```

使用这种方法能够让我们在不离开 Container 的状态下操作容器里面的值。Container 里的值传递给 map 函数之后，就可以任我们操作；操作结束后，为了防止意外再把它放回它所属的 Container。这样做的结果是，我们能连续调用 map，运行任何我们想运行的函数，甚至还可以改变值的类型。

```javascript
// 连续调用 map
Container.of('bombs').map(concat(' away')).map(_.prop('length'));
// => Container(10)
```

换个角度思考，如果我们能够一直调用 map，那它其实就是一个 composition。这引申出 functor 的定义：

**functor 是实现了 map 函数并遵守一些特定规则的容器类型。**

把值放进一个容器，而且只能使用 map 来处理它。当 map 一个函数的时候，我们请求容器来运行这个函数。这样我们可以得到对于**函数运用的抽象**。

---

**Maybe _(one kind of functors)_ 空值检查**

首先我们创建一个 Maybe：

```javascript
const Maybe = function(x) {
  this.__value = x;
}

Maybe.of = x => new Maybe(x);

Maybe.prototype.isNothing = function() {
  return (this.__value === null || this.__value === undefined);
}

Maybe.prototype.map = function(f) {
  return this.isNothing() ? Maybe.of(null) : Maybe.of(f(this.__value));
}
```

然后就可以这样调用：

```javascript
Maybe.of('Malkovich Malkovich').map(match(/a/ig));
// => Maybe(['a', 'a'])

Maybe.of(null).map(match(/a/ig));
// => Maybe(null)

Maybe.of({ name: 'Boris' }).map(prop('age')).map(add(10));
// => Maybe(null)

Maybe.of({ name: 'Dinah', age: 14 }).map(prop('age')).map(add(10));
// => Maybe(24)
```

Maybe 会先检查自己的额值是否为空，然后才调用传进来的函数。这样我们在使用 map 的时候就能避免麻烦的空值了。

上面调用的例子中，点记法 _(dot notation syntax)_ 已经足够函数式，但如果要追求 piontfree，map 可以用 curry 函数的方式来代理任何 functor：

```javascript
// map :: Functor f => (a -> b) -> f a -> f b
const map = curry(function(f, any_functor_at_all) {
  return any_functor_at_all.map(f);
});
```

Maybe 最常用于可能会无法成功返回结果的函数中：

```javascript
// safeHead :: [a] -> Maybe(a)
const safeHead = xs => Maybe.of(xs[0]);

const streetName = compose(
  map(prop('street')),
  safeHead,
  prop('addresses'),
);

streetName({ addresses: [] });
// => Maybe(null)

streetName({ addresses: [ { street: 'Shady Ln.', number: 4201 } ] });
// => Maybe('Shady Ln.')
```

---

**释放容器里的值 _the maybe helper function_**

产生作用 _effects_ 的函数，不管是发送 JSON 数据，还是在屏幕上打印东西，还是更改文件系统，都要有一个结束。但我们无法通过 return 把输出传递到外部世界，必须要运行这样或那样的函数才能传递出去。

应用程序所做的工作就是获取、更改和保存数据直到不再需要它们，对数据做这些操作的函数有可能被 map 调用，这样数据就可以不用离开容器。有一种常见的错误是试图以各种方法删除 Maybe 里面的值，从而能得到一种确定性。但要明白的是，我们的代码，像薛定谔的猫一样，在某个特定时间点有两种状态，而且应该保持这种状态不变直到最后一个函数为止。这样，哪怕代码有很多逻辑性分支，也能保证一种线性的工作流。

但对于容器里的值来说，还是可以有个出口。如果我们想返回一个自定义的值然后还能继续执行后续的代码，可以借助于一个帮助函数 maybe：

```javascript
// maybe :: b -> (a -> b) -> Maybe a -> b
const maybe = curry(function(x, f, m) {
  return m.isNothing ? x : f(m.__value);
});

// getTwenty :: Account -> String
const getTwenty = compose(
  maybe('You are broke!', finishTransaction),
  withdraw(20),
);

getTwenty({ balance: 200.00 });
// => "Your balance is $180.00"

getTwenty({ balance: 10.00 });
// => "You are broke!"
```

这样就可以要么返回一个静态值，与 finishTransaction 返回值的类型一致，要么继续在没有 Maybe 的情况下完成交易。maybe 使我们得以避免普通 map 那种命令式的 if/else 语句：if (x !== null) { return f(x) }

真正的 Maybe 实现会把它分为两种类型：一种是非空值，一种是空值。这种实现允许我们遵守 map 的 parametricity 特性，因此 null 和 undefined 能够依然被 map 调用，functor 里面的值所需的那种普遍性条件也能得到满足。所以会经常看见 Some(x) / None 或者 Just(x) / Nothing 这样的容器类型在做空值检查，而不是 Maybe。

---

**Either _(one kind of functor)_ 纯错误处理**

```javascript
const Left = function(x) {
  this.__value = x;
}

Left.of = x => new Left(x);

Left.prototype.map = function(f) {
  return this;
}

const Right = function(x) {
  this.__value = x;
}

Right.of = x => new Right(x);

Rgiht.prototype.map = function(f) {
  return Right.of(f(this.__value));
}
```

Left 和 Right 被称之为 Either 的抽象类型的两个子类。Left 无视 map 的请求，Right 像一个 Container，但在这里 Left 有能力在内部嵌入一个错误信息。

```javascript
const moment = require('moment');

// getAge :: Date -> User -> Either(String, Number)
const getAge = curry((now, user) => {
  const birthdate = moment(user.birthdate, 'YYYY-MM-DD');
  if (!birthdate.isValid()) return Left.of('Birth date could not be parsed');
  return Right.of(now.diff(birthdate, 'years'));
});

getAge(moment(), { birthdate: '2005-12-12' });
// => Right(14)

getAge(moment(), { birthdate: 'balloons!' });
// => Left("Birth date could not be parsed")
```

当返回一个 Left 的时候就直接让程序短路。有一件事要注意，这里返回的是 Either(String, Number)，意味着我们这个 Either 左边是 String，右边 _right_ 是 Number。这个类型签名不够正式，因为我们没有定义一个真正的 Either 父类，但还是能从这个类型申明了解到不少情况。

```javascript
// fortune :: Number -> String
const fortune = compose(concat("If you survive, you will be "), add(1));

// zoltar :: User -> Either(String, _)
const zoltar = compose(map(console.log), map(fortune), getAge(moment()));

zoltar({ birthdate: '2005-12-12' });
// "If you survive, you will be 10"
// Right(undefined)

zoltar({ birthdate: 'balloons!' });
// Left("Birth date could not be parsed")
```

如果 birthdate 合法，正常运行；不合法，收到一个 Left 包裹着错误消息。一种平静温和的方式抛错。这个例子中我们根据 birthdate 的合法性来空值代码的逻辑分支，同时又让代码从右到左的直线运行。另外我们在 Right 分支的类型签名中使用 _ 表示一个应该忽略的值。

尽管 fortune 使用了 Either，但它对每一个 functor 要做什么毫不知情。当一个函数在调用的时候，如果被 map 包裹，那么它就会从一个非 functor 函数转换成为以一个 functor 函数。我们把这个过程叫做 **lift**。一般情况下，普通函数更适合操作普通的数据类型而不是容器类型，在必要的时候再通过 lift 变为合适的容器去操作容器类型。这样做的好处是能得到更简单、重用性更高的函数，它们能过随需求而变，兼容任意 functor。

Either 并不仅仅只对合法性检查这种一般性的错误有效，对一些更严重、能够终端程序执行错误比如文件丢失或者 socket 连接断开等，Either 同样效果显著。

Either 不仅仅可以作为一个错误消息的容器，它表示了逻辑或，体现了范畴学里 coproduct 的概念。

---

**the either helper function**

```javascript
// either :: (a -> c) -> (b -> c) -> Either a b -> c
const either = curry(function(f, g, e) {
  switch(e.constructor) {
    case Left: return f(e.__value);
    case Right; return g(e.__value);
  }
});

// zoltar :: User -> _
cconst zoltar = compose(console.log, either(id, fortune), getAge(moment()));

zoltar({ birthdate: '2005-12-12' })
// "If you survive, you will be 10"
// undefined

zoltar({ birthdate: 'balloons!' });
// "Birth date could not be parsed"
// undefined
```

---

**Handle effects**

```javascript
// getFromStorage :: String -> (_ -> String)
const getFromStorage = function(key) {
  return function() {
    return localStorage[key];
  }
}
```

把会产生副作用的函数包裹在另一个函数里把它变得看起来像一个纯函数。有了这个包裹函数 warpper，同一个输入总能返回同一个输出：一个从 localStorage 里取出某个特定元素的函数。

引入 IO：

```javascript
const IO = function(f) {
  this.__value = f;
}

IO.of = function(x) {
  return new IO(function() {
    return x;
  });
}

IO.prototype.map = function(f) {
  return new IO(compose(f, this.__value));
}
```

IO 的 `__value` 总是一个函数。把非纯执行动作放在包裹函数里，目的是延迟执行这个非纯动作。就这一点而言，我们认为 IO 包含的是被包裹执行动作的返回值，而不是包裹函数本身。

```javascript
// ioWindow :: IO Window
const ioWindow = new IO(() => window);

ioWindow.map(win => win.innerWidth);
// IO(1430)

ioWindow
  .map(prop('location'))
  .map(prop('href'))
  .map(split('/'));
// IO(['http:', '', 'localhost:8000', 'blog', 'posts'])

// $ :: String -> IO [DOM]
const $ = selector => new IO(() => document.querySelectorAll(selector));

$('#myDiv')
  .map(head)
  .map(div => div.innerHTML)
// IO('I am some inner html')
```

```javascript
// url :: IO String
const url = new IO(() => window.location.href);

// toPairs = String => [[String]]
const toPairs = compose(
  map(split('=')),
  split('&'),
);

// params :: String -> [[String]]
const params = compose(
  toPairs,
  last,
  split('?'),
);

// findParam :: String -> IO Maybe [String]
const findParam = key => map(
  compose(
    Maybe.of,
    filter(
      compose(
        eq(key),
        head,
      ),
    ),
    params,
  ),
  url,
);

// 调用 __value() 来运行
findParam('searchTerm').__value();
// Maybe(['searchTerm', 'wafflehouse'])
```

这个例子汇总我们把 url 包裹在一个 IO 里，然后传给了调用者。同时我们也压栈了容器，创建了一个 IO(Maybe([x]))，最后这个栈有三层 functor。

IO 的 `__value` 并不是它包含的值，也不是像两个下划线暗示是一个私有属性。`__value` 是手榴弹的弹栓，只应该被调用者以最公开的方式拉动。为了提醒用户它的变化无常，我们把它重命名为 unsafePerformIO 看看。

```javascript
const IO = function(f) {
  this.unsafePerformIO = f;
}

IO.prototype.map = function(f) {
  return new IO(compose(f, this.unsafePerformIO));
}
```

现在调用代码变成了 `findParam('searchTerm').unsafePerformIO()`，这样对于程序的用户来说，不能更直白了。

---

**异步任务**

没人喜欢 callback hell。借用 Quildreen Motta 的 [Folktale](https://github.com/origamitower/folktale) 里面的 `Data.Task` 来解释一下。

```javascript
const fs = require('fs');

// callback style
// readFile :: String -> Task(Error, JSON)
const readFile = function(filename) {
  return new Task(function(reject, result) {
    fs.readFile(filename, 'utf-8', function(err, data) {
      err ? reject(err) : result(data);
    });
  });
};

readFile('metamorphosis').map(split('\n')).map(head);

// callback style
// getJSON :: String -> {} -> Task(Error, JSON)
const getJSON = curry(function(url, params) {
  return new Task(function(reject, result) {
    $.getJSON(url, params, result).fail(reject);
  });
});

getJSON('/video', { id: 10 }).map(prop('title'));
```

里面的 Task 的 map 函数可以类比于 Promise 的 then。Task 的工作方式和 IO 毫无差别，都是把对未来的操作放在一个时间胶囊。

```javascript
// blogPage :: Posts -> HTML
const blogPage = Handlebars.compile(blogTemplate);

// renderPage :: Posts -> HTML
const renderPage = compose(
  blogPage,
  sortBy('date'),
);

// blog :: Params -> Task(Error, HTML)
const blog = compose(
  map(renderPage),
  getJSON('/posts'),
);

// Impure calling
blog({}).fork(
  function(error) { $('#errlr').html(error.message) },
  function(page) { $('#main').html(page) }
);
$('#spinner').show();
```

调用 fork 来运行 Task，这种机制与 unsafePerformIO 类似，不同的地方在于 fork 是 fork 一个子进程运行它接收到的参数代码，其他部分的执行不受影响，主线程也不会阻塞。

调用 fork 后，Task 就会去找一些文章，渲染到页面上，与此同时，我们在页面上现实一个 spinner，因为 fork 不会等收到了响应后才执行它后面的代码。最后，我们要么把文章展示在页面上，要么就显示一个出错信息。

这段代码我们只需要从下读到上，从右到左就能理解代码，即便这段程序实际上会在执行过程中跳来跳去。这种方式是的阅读和理解应用程序的代码比那种要在各种回调和错误处理代码之间跳跃的方式要容易的多。你也会发现 Task 里面包含了 Either，为了能处理将来可能出现的错误，它必须得这么做，因为普通的控制流在异步世界里不适用。

再看一个例子：

```javascript
// Postgres.connect :: Url -> IO DbConnection
// runQuery :: DbConnection -> ResultSet
// readFile :: String -> Task Error String

// dbUrl :: Config -> Either Error Url
const dbUrl = function(c) {
  return (c.uname && c.pass && c.host && c.db)
    ? Right.of(`db:pg://${c.name}:${c.pass}@${c.host}5432/${c.db}`)
    : Left.of(Error('Invalid Config'));
}

// connectDb :: Config -> Either Error (IO DbConnection)
const connectDb = compose(
  map(Postgre.connect),
  dbUrl,
);

// getConfig :: Filename -> Task Error (Either Error (IO DbConnection))
const getConfig = compose(
  map(compose(
    connectDb,
    JSON.parse,
  )),
  readFile,
);

// Impure calling
getConfig('db.json').fork(
  logErr('could not read file'),
  either(console.log, map(runQuery)),
);
```

在这个例子中我们在 readFile 成功的代码分支里利用了 Either 和 IO。Task 处理一步读取文件这一操作的不纯性，但是验证 config 的合法性以及连接数据库则分别使用了 Either 和 IO。所以我们依然在同步的和所有事物打交道。

---

**一点理论**

functor 概念来自于范畴学，并满足一些定律。

```javascript
// identity
map(id) === id;

// composition
compose(map(f), map(g)) === map(compose(f, g));
```

在范畴学中，functor 接受一个范畴的对象和态射(morphism)，然后把它们映射(map)到另一个范畴里去。根据定义，这个新范畴一定会有一个单位元(identity)，也一定能够组合态射。

你可以把范畴想象成一个有着多个对象的网络，对象之间靠态射连接。那么 functor 可以把一个范畴映射到另外一个，而且不会破坏原有的网络。如果一个对象 a 属于源范畴 C，那么通过 functor F 把 a 映射到目标范畴 D 上之后，就可以使用 F a 来指代 a 对象。

比如，Maybe 就是把类型和函数的范畴映射到这样一个范畴：即每个对象都有可能不存在，每个态射都有空值检查的范畴。这个结果在代码中的实现方式是用 map 包裹每一个函数，用 functor 包裹每一个类型。这样就能保证每个普通的类型和函数都能在新环境下继续组合使用。从技术上讲，代码中的 functor 实际上是把范畴映射到了一个包含类型和函数的子范畴(sub category)上，使得这些 funtor 成为了一种新的特殊的 endofunctor。

```
a -> f -> b
a -> F.of -> F a
b -> F.of -> F b
F a -> map(f) -> F b
```

通过上图可以发现它符合交换律。这种形式化给我们原则性的方式去思考代码，无须分析和评估每一个单独的场景，直接套用应用公式即可。

```javascript
// topRoute :: String -> Maybe(String)
const topRoute = compose(
  Mabe.of,
  reverse,
);

// bottomRoute :: String -> Maybe(String)
const bottomRoute = compose(
  map(reverse),
  Maybe.of,
);


topRoute('hi');
// Maybe('ih')

bottomRoute('hi');
// Maybe('ih')
```

上面代码转换为图例：

```
'hi' -> reverse -> 'ih'
'hi' -> Maybe.of -> Maybe('hi')
'ih' -> Maybe.of -> Maybe('ih')
Maybe('hi') -> map(reverse) -> Maybe('ih')
```

根据这种特性，我们可以立即理解代码或重构。

functor 也能嵌套使用：

```javascript
const nested = Task.of([
  Right.of('pillows'),
  Left,of('no sleep for you'),
]);

map(map(map(toUpperCase)), nested);
// Task([Right('PILLOWS'), Left('no sleep for you')])

// 1st map: map on Task.of, get [Right.of('pillows'), Left.of('no sleep for you')]
// 2nd map: map on Array, iterate Right.of('pillows'), Left.of('no sleep for you')
// 3rd map: map on Either, Right('pillow'), Left('no sleep for you')
```

我们必须要 map(map(map(f))) 才能最终运行函数，如果不想这么做，可以组合 functor。

```javascript
const Compose = function(f_g_x) {
  this.getCompose = f_g_x;
}

Compose.prototype.map = function(f) {
  return new Compose(map(map(f), this.getCompose));
}

const tmd = Task.of(Maybe.of('Rock over London'));
const ctmd = new Compose(tmd);

map(concat(', rock on, Chicago'), ctmd);
// Compose(Task(Maybe('Rock over London, rock on, Chicago')))

ctmd.getCompose;
// Task(Maybe('Rock over London, rock on, Chicago'))
```

functor 组合是符合结合律的，之前我们定义的 Container 实际上是一个叫 Identity 的 functor。identity 和可结合的组合也能产生一个范畴，这个额头书的范畴的对象是其他范畴，态射是 functor。

## 9) Monad

**pointed functor**

之前我们创建的容器类型的 of 方法，不是用来避免使用 new 关键字的，而是用来把值放到默认最小化上下文(default minimal context)中。of 没有真正的取代构造器，而是一个我们称之为 pointed 的重要接口的一部分。

pointed functor 是实现了 of 方法的 functor。

这里的关键是把任意值丢到容器里然后开始到处使用 map 的能力。

```javascript
IO.of('tetris').map(concat(' master'));
// IO("tetris master")

Maybe.of(123).map(add(1));
// Maybe(124)

Task.of([{ id: 2 }, { id: 3 }]).map(prop('id'));
// Task([2, 3])

Either.of('The past, present and future walk into a bar...').map(concat('it was tense.'))
// Right("The past, present and future walk into a bar...it was tense.")
```

IO 和 Task 的构造器接受一个函数作为参数，而 Maybe 和 Either 的构造器可以接受任意值。实现这种接口的动机是，我们希望能够有一种通用一致的方式往 functor 里填值，而且中间不会涉及到复杂性，也不会涉及到对构造器的特定要求。“默认最小化上下文”这个术语可能不够精确，但是却很好的传达了这种理念：我们希望容器类型里的任意值都能发生 lift，然后像所有的 functor 那样再 map 出去。

每个 functor 都要有一种把值放进去的方式，但对 Either 来说，它的方式是 new Right(x)。我们为 Right 定义 of 的原因是，如果一个类型容器可以 map，那它就应该 map。

---

**混合比喻**

```javascript
// readFile :: String -> IO String
const readFile = function(filename) {
  return new IO(function() {
    return fs.readFileSync(filename, 'utf-8');
  });
};

// print :: String -> IO String
const print = function(x) {
  return new IO(function() {
    console.log(x);
    return x;
  });
}

// cat :: IO (IO String)
const cat = compose(map(print), readFile);

// catFirstChar :: String -> IO (IO String)
const catFirstChar = compose(map(map(head)), cat);

cat('.git/config')
// IO(IO("[core]\nrepositoryformatversion = 0]n"))

catFirstChar('.git/config')
// IO(IO("["))
```

我们得到了一个前套的 IO，要使用它，我们必须 map(map(f))，要观察它的作用，我们必须 unsafePerformIO().unsafePerformIO()。

另一个例子：

```javascript
// safeProp :: Key -> {Key : a} -> Maybe a
const safeProp = curry(function(x, obj) {
  return new Maybe(obj[x]);
});

// safeHead :: [a] -> Maybe a
const safeHead = safeProp(0);

// firstAddressStreet :: User -> Maybe (Maybe (Maybe Street))
const firstAddressStreet = compose(
  map(map(safeProp('street'))),
  map(safeHead),
  safeProp('addresses'),
);

firstAddressStreet(
  {
    address: [
      {
        street: {
          name: 'Mulburry',
          number: 8402,
        },
        postcode: 'WC2N',
      }
    ]
  }
);
// Maybe(Maybe(Maybe({ name: 'Mulburry', number: 8402 })))
```

这种嵌套 functor 的模式会时不时出现，而且是 monad 的主要使用场景。当碰到这种情况时，我们可以使用一个叫做 `join` 的方法。

```javascript
const mmo = Maybe.of(Maybe.of('ok'));
// Maybe(Maybe('ok'))

mmo.join();
// Maybe('ok')

const ioio = IO.of(IO.of('pizza'));
// IO(IO('pizza'))

ioio.join();
// IO('pizza')

const ttt = Task.of(Task.of(Task.of('some tasks')));
// Task(Task(Task('some tasks')))

ttt.join()
// Task(Task('some tasks'))
```

如果有两层相同类型的嵌套，那么就可以使用 join 把它们压扁道一块儿去。

**monad 是可以变扁的(flattern) pointed functor**

一个 functor，只要它定义了一个 join 方法和一个 of 方法，并遵守一些定律，那么它就是一个 monad。

我们来为 Maybe 定义一个 join 方法：

```javascript
Maybe.prototype.join = function() {
  return this.isNothing() ? May.of(null) : this.__value;
}
```

下面看下 Maybe.join 的实际应用：

```javascript
// join :: Monad m => m (m a) -> m a
const join = function(mma) { return mma.join(); }

// firstAddressStreet :: User -> Maybe Street
const firstAddressStreet = compose(
  join,
  map(safeProp('street')),
  join,
  map(safeHead),
  safeProp('addresses'),
);

firstAddressStreet(
  {
    address: [
      {
        street: {
          name: 'Mulburry',
          number: 8402,
        },
        postcode: 'WC2N',
      }
    ]
  }
);
// Maybe({ name: 'Mulburry', number: 8402 })
```

只要遇到嵌套的 Maybe，就加一个 join。

下面我们再试试 IO 的 join：

```javascript
IO.prototype.join = function() {
  return this.unsafePerformIO();
}

// log :: a -> IO a
const log = function(x) {
  return new IO(function() { console.log(x); return x; })
}

// setStyle :: Selector -> CSSProps -> IO DOM
const setStyle = curry(function(sel, props) {
  return new IO(function() { return jQuery(sel).css(props); });
});

// getItem :: String -> IO String
const getItem = function(key) {
  return new IO(function() { return localStorage.getItem(key); })
}

// applyPreferences :: String -> IO DOM
const applyPreferences = compose(
  join,
  map(setStyle('#main')),
  join,
  map(log),
  map(JSON.parse),
  getItem,
);

applyPreferences('preferences').unsafePerformIO();
// Object {backgroundColor: 'green'}
// <div style="background-color: 'green'" />
```

getItem 返回了一个 IO String，所以可以直接用 map 来解析它。log 和 setStyle 返回的都是 IO，所以必须要用 join 来保证里面的嵌套处于控制之中。

---

**chain 函数**

从上面的例子可以看出总是在紧跟着 map 后调用 join，我们可以把这个行为抽象到一个叫做 chain 的函数里：

```javascript
const chain = curry(function(f, m) {
  return m.map(f).join();
})
```

chain 在其他函数式编程语言中又称为 `>>=`(读作 bind) 或者 `flatMap`，同一个概念，不同名称。flatMap 是最准确的名称，但后续还是会继续用 chain。

我们用 chain 来重构上面的例子：

```javascript
// map/join
const firstAddressStreet = compose(
  join,
  map(safeProp('street')),
  join,
  map(safeHead),
  safeProp('addresses'),
);

// chain
const firstAddressStreet = compose(
  chain(safeProp('street')),
  chain(safeHead),
  safeProp('addresses'),
)

// map/join
const applyPreferences = compose(
  join,
  map(setStyle('#main')),
  join,
  map(log),
  map(JSON.parse),
  getItem,
);

// chain
const applyPreferences = compose(
  chain(setStyle('#main')),
  chain(log),
  map(JSON.parse),
  getItem,
);
```

chain 的能力不止于此，因为可以轻松嵌套多个作用，所以我们能以一种纯函数式的方式来表示序列 `sequence` 和变量赋值 `variable assignment`。

```javascript
// getJSON :: Url -> Params -> Task JSON
// querySelector :: Selector -> IO DOM

getJSON('/authenticate', { username: 'stele', password: 'crackers' })
  .chain(function(user) {
    return getJSON('/friends', { user_id: user.id });
  });
// Task([{ name: 'Seimith', id: 14 }, { name: 'Ric', id: 39 }])

querySelector('input.username').chain(function(uname) {
  return querySelector('input.email').chain(function(email) {
    return IO.of(
      `Welcome ${uname.value} prepare for span at ${email.value}`
    );
  });
});
// IO('Welcome Olivia prepare for span at olivia@somesite.com')

Maybe.of(3).chain(function(three) {
  return Maybe.of(2).map(add(three));
});
// Maybe(5);

Maybe.of(null).chain(safeProp('address')).chain(safeProp('street'));
// Maybe(null);
```

chain 可以自动从任意类型的 map 和 join 衍生出来，就像这样：t.prototype.chain = function(f) { return this.map(f).join(); }。如果 chain 是简单的通过结束调用 of 后把值放回容器这种方式定义的，那么会造成一个有趣的后果，即可以从 chain 衍生出一个 map。同样，我们还可以用chain(id) 定义 join。

总之记住，返回的如果是“普通”值就用 map，如果是 functor 就用 chain。

---

**炫耀**

这种容器编程风格优势也能造成困惑，我们不得不努力理解一个值到底嵌套了基层容器，或者需要用 map 还是 chain。

我们先看一下 chain 的威力：

```javascript
// readFile :: Filename -> Either String (Future Error String)
// httpPost :: String -> Future Error JSON

// upload :: String -> Either String (Future Error JSON)
const upload = compose(
  map(chain(httpPost('/uploads'))),
  readFile,
)

// 和命令式方式对比一下
const upload = function(filename, callback) {
  if (!filename) {
    throw 'You need a filename!';
  } else {
    readFile(filename, function(err, contents) {
      if (err) throw err;
      httpPost(contents, function(err, json) {
        if (err) throw err;
        callback(json);
      });
    });
  }
};
```

---

**理论**

```
// 结合律
compose(join, map(join)) == compose(join, join)

// 用等式推导：
// 合并里面的两个 M
1. M(M(M a)) -> map(join) -> M(M a) -> join -> M a

// 合并外面的两个 M
2. M(M(M a)) -> join -> M(M a) -> join -> M a
```

```
// 同一律 (M a)
compose(join, of) == compose(join, map(of)) == id

// 用等式推导：
// M a -> of -> M(M a) -> join -> M a
// M a -> map(of) -> M(M a) -> join -> M a
// M a -> id -> M a
```

```javascript
const mcompose = function(f, g) {
  return compose(chain(f), chain(g));
}

// 左同一律
mcompose(M, f) == f

// 右同一律
mcompose(f, M) == f

// 结合律
mcompose(mcompose(f, g), h) == mcompose(f, mcompose(g, h))
```

## 10) Applicative Functor

想象一个场景，有两个同类型的 functor，想把这两者作为一个函数的两个参数传递过去来调用这个函数。比如：

```javascript
// not gonna work
add(Container.of(2), Container.of(3))
// NaN

// Use map
const container_of_add_2 = map(add, Container.of(2));
// Container(add(2))

Container.of(2).chain(function(two) {
  return Container.of(3).map(add(two));
});
```

上面代码存在的问题是，所有代码都只会在前一个 monad 执行完毕之后才执行。

这里引入 `ap` 函数。能够把一个 functor 的函数值应用到另一个 functor 的值上。

```javascript
Container.of(add(2)).ap(Container.of(3));
// Container(5)

// all together now
Container.of(2).map(add).ap(Container.of(3));
// Container(5)
```

可以这样定义一个 ap 函数：

```javascript
Container.prototype.ap = function(other_container) {
  return other_container.map(this.__value);
}
```
