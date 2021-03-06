# ES6对对象的扩展
#### 1、属性和方法的简洁表示法
```javascript
//属性：
  const baz = {foo};
  //等同于
  const baz = {foo: foo};
//方法：
  const o = {
    method() {
      return "Hello!";
    }
  };

  // 等同于

  const o = {
    method: function() {
      return "Hello!";
    }
  };
```
简洁写法的属性名总是字符串，这会导致一些看上去比较奇怪的结果。
```javascript
const obj = {
  class () {}
};
// 等同于
var obj = {
  'class': function() {}
};
上面代码中，class是字符串，所以不会因为它属于关键字，而导致语法解析报错。
某个方法的值是一个 Generator 函数，前面需要加上星号。
const obj = {
  * m() {
    yield 'hello world';
  }
};
```
#### 2、属性名表达式
JavaScript 定义对象的属性，有两种方法。
```javascript
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```
ES6 允许字面量定义对象时，可表达式作为对象的属性名，表达式还可以用于定义方法名，即把表达式放在方括号内。<br>
注意，属性名表达式与简洁表示法，不能同时使用，会报错。
```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};

let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};

obj.hello() // hi

//注意，属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串[object Object]，这一点要特别小心。
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
//上面代码中，[keyA]和[keyB]得到的都是[object Object]，所以[keyB]会把[keyA]覆盖掉，而myObject最后只有一个[object Object]属性。
```
#### 3、方法的 name 属性 
函数的name属性，返回函数名。<br>
对象方法也是函数，因此也有name属性。<br>
有特殊情况。
```javascript
const person = {
  sayName() {
    console.log('hello!');
  },
};

person.sayName.name   // "sayName"
```
#### 4、Object.is()
ES5 比较两个值是否相等，只有两个运算符：相等运算符（==）和严格相等运算符（===）。<br>
它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。<br>
JavaScript 缺乏一种运算，在所有环境中，只要两个值是一样的，它们就应该相等。<br>
ES6 提出同值相等算法，用来解决这个问题。<br>
Object.is就是部署这个算法的新方法，它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。
```javascript
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false

+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```
#### 5、Object.assign()
Object.assign方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）
```javascript
const target = { a: 1 };

const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
//Object.assign方法的第一个参数是目标对象，后面的参数都是源对象
//如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
//如果只有一个参数，Object.assign会直接返回该参数。
//如果只有一个参数，该参数不是对象，则会先转成对象，然后返回。
//如果只有一个参数，由于undefined和null无法转成对象，所以如果它们作为参数，就会报错。
//如果非对象参数出现在源对象的位置（即非首参数），那么处理规则有所不同。首先，这些参数都会转成对象，如果无法转成对象，就会跳过。这意味着，如果undefined和null不在首参数，就不会报错。
//其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。
//属性名为 Symbol 值的属性，也会被Object.assign拷贝。
```
##### 注意点：
1、浅拷贝：Object.assign方法实行的是浅拷贝，而不是深拷贝。<br>
2、同名属性的替换：对于这种嵌套的对象，一旦遇到同名属性，Object.assign的处理方法是替换，而不是添加。<br>
3、处理数组：Object.assign可以用来处理数组，但是会把数组视为对象。
```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```
4、取值函数的处理：Object.assign只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制。
#### 6、属性的可枚举性和遍历
对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。<br>
Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象。
```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,   //true：”可枚举性“
//    configurable: true
//  }

有四个操作会忽略enumerable为false（不可枚举性）的属性。

for...in循环：只遍历对象自身的和*继承的*可枚举的属性。
Object.keys()：返回对象自身的所有可枚举的属性的键名。
JSON.stringify()：只串行化对象自身的可枚举的属性。
Object.assign()： 只拷贝对象自身的可枚举的属性。

//操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，尽量不要用for...in循环，而用Object.keys()代替。
```
#### 7、__proto__属性，Object.setPrototypeOf()，Object.getPrototypeOf()
##### （1）__proto__属性
__proto__属性（前后各两个下划线），用来读取或设置当前对象的prototype对象。目前，所有浏览器（包括 IE11）都部署了这个属性。<br>
该属性没有写入 ES6 的正文，而是写入了附录，原因是__proto__前后的双下划线，说明它本质上是一个内部属性，而不是一个正式的对外的 API，只是由于浏览器广泛支持，才被加入了 ES6。标准明确规定，只有浏览器必须部署这个属性，其他运行环境不一定需要部署，而且新的代码最好认为这个属性是不存在的。因此，无论从语义的角度，还是从兼容性的角度，都不要使用这个属性，而是使用下面的Object.setPrototypeOf()（写操作）、Object.getPrototypeOf()（读操作）、Object.create()（生成操作）代替
##### （2）Object.setPrototypeOf()
Object.setPrototypeOf方法的作用与__proto__相同，用来设置一个对象的prototype对象，返回参数对象本身。<br>
它是 ES6 正式推荐的设置原型对象的方法。
```javascript
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40
```
##### （3）Object.getPrototypeOf()
该方法与Object.setPrototypeOf方法配套，用于读取一个对象的原型对象。
#### 8、super 关键字
this关键字总是指向函数所在的当前对象；<br>
ES6 又新增了另一个类似的关键字super，指向当前对象的原型对象。
```javascript
const proto = {
  foo: 'hello'
};

const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find() // "hello"

//super关键字表示原型对象时，只能用在对象的方法之中，用在其他地方都会报错。
// 报错 super用在属性里面
const obj = {
  foo: super.foo
}

// 报错 只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法。
const obj = {
  foo: () => super.foo
}

// 报错 只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法。
const obj = {
  foo: function () {
    return super.foo
  }
}
```
```javascript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};

const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);

obj.foo() // "world"
//上面代码中，super.foo指向原型对象proto的foo方法，但是绑定的this却还是当前对象obj，因此输出的就是world。
```
#### 9、Object.keys()，Object.values()，Object.entries()
ES5 引入了Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历的（enumerable）属性的键名。<br>
ES6 引入了跟Object.keys配套的Object.values和Object.entries，作为遍历一个对象的补充手段，供for...of循环使用。<br>
Object.values方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值。<br>
Object.entries方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组。
```javascript
let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```
#### 10、对象的扩展运算符
对象的解构赋值用于从一个对象取值，相当于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。
```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```
对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。
```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

*文章内容参考阮一峰老师的《ECMAScript 6 入门》文章内容，如有侵权联系删除*











