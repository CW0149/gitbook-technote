# this和对象原型

<!-- toc -->

## this

### 为什么需要this(Why)

在编程中，经常需要在函数间传递上下文对象，如果使用参数显示传递方式，随着使用模式越来越复杂，代码会变得越来越乱。this 提供了一种优雅的方式来隐式“传递”一个对象引用，帮助我们将 API 设计得更加简洁并且易于复用。

### this是什么(What)

当一个函数被调用时，会创建一个活动记录(有时候也称为执行上下文)。这个记录会包含函数在哪里被调用(调用栈)、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。你可以通过在控制台给代码添加断点来研究调用栈。

this 既不指向函数自身也不指向函数的词法作用域，它是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用(调用位置)。调用位置在当前正在执行的函数的前一个调用中。

### this绑定规则(How)

#### 隐式绑定

若函数引用有上下文对象，则函数调用中的this会被绑定到这个上下文对象。

**隐式丢失**

```
function foo() {
	console.log( this.a );
}
var obj = {
	a: 2,
	foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"

JavaScript 环境中内置的 setTimeout() 函数实现和下面的伪代码类似:
function setTimeout(fn,delay) { // 等待 delay 毫秒
	fn(); // <-- 调用位置!
}
```

#### 显式绑定

使用call(...)和apply(...)方法，可以将this指定绑定到特定对象上。

第三方库的许多函数，以及 JavaScript 语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为“上下文”(context)，其作用和 bind(..) 一样，确保你的回调函数使用指定的 this。

```
// 调用 foo(..) 时把 this 绑定到
obj [1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

#### new绑定

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

new绑定比显式绑定优先级更高。

#### 默认绑定

若不符合上述规则，this会采用默认绑定。this默认绑定在非严格模式下绑定到全局对象，严格模式下值为undifined。

### this绑定实践

this具有默认绑定，函数调用时可能改变this的绑定。

#### 判断this的绑定

1. 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。
     var bar = new foo()
2. 函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是指定的对象。
     var bar = foo.call(obj2)
3. 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this 绑定的是那个上下文对象。
     var bar = obj1.foo()
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。
     var bar = foo()

#### 注意点

有些调用可能在无意中使用默认绑定规则。如果想“更安全”地忽略 this 绑定，你可以使用一个 DMZ 对象，比如 ø = Object.create(null)，以保护全局对象。

例如，若将 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。更安全的做法是使用`Object.create(null)`创建一个不会创建Object.prototype委托的干净的空对象，将其作为call、apply的首个参数。

#### 软绑定

硬绑定可以把 this 强制绑定到指定的对象(除了使用 new 时)，但是硬绑定会大大降低函数的灵活性-使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this。

如果可以给默认绑定指定一个全局对象和 undefined 以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显式绑定修改 this 的能力，这可以解决隐式丢失的问题。

下面是softBind的实现。softBind(..) 的其他原理和 ES5 内置的 bind(..) 类似。它会对指定的函数进行封装，首先检查调用时的 this，如果 this 绑定到全局对象或者 undefined，那就把指定的默认对象 obj 绑定到 this，否则不会修改 this。

```
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this;
		// 捕获所有 curried 参数
		var curried = [].slice.call( arguments, 1 );
		var bound = function() {
			return fn.apply(
				(!this || this === (window || global)) ?
				obj : this
				curried.concat.apply( curried, arguments )
			);
		};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}

// 示例
function foo() {
	console.log("name: " + this.name);
}
var obj = { name: "obj" }, obj2 = { name: "obj2" }, obj3 = { name: "obj3" };
var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj
obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2 <---- 看!!!
fooOBJ.call( obj3 ); // name: obj3 <---- 看!
setTimeout( obj2.foo, 10 ); // name: obj <---- 应用了软绑定
```

#### 箭头函数

箭头函数不使用 this 的四种标准规则，而是根据外层(函数或者全局)作用域来决定 this。箭头函数会继承外层函数调用的 this 绑定(无论 this 绑定到什么)。这其实和 ES6 之前代码中的 self = this 机制一样。

在同一个函数或者同一个程序中混合使用这两种风格通常会使代码更难维护，并且可能也会更难编写。如果你经常编写 this 风格的代码，但是绝大部分时候都会使用 self = this 或者箭头函数来否定 this 机制，那你或许应当:

1. 只使用词法作用域并完全抛弃错误this风格的代码;
2. 完全采用 this 风格，在必要时使用 bind(..)，尽量避免使用 self = this 和箭头函数。


## 对象

### 定义

对象可以通过两种形式定义:声明(文字)形式和构造形式。这两种形式生成的对象是一样的。唯一的区别是，在文字声明中你可以添加多个键 / 值对，但是在构造形式中你必须逐个添加属性。字面形式更常用，不过有时候构造形式可以提供更多选项。

```
对象的文字语法大概是这样:
var myObj = {
	key: value
	// ...
};
构造形式大概是这样:
var myObj = new Object();
myObj.key = value;
```

### 类型

#### 主要类型

在JavaScript中一共有六种主要类型(术语是“语言类型”)：

- string
- number
- boolean
- null
- undefined
- object

null 有时会被当作一种对象类型，但是这其实只是语言本身的一个 bug，即对 null 执行 typeof null 时会返回字符串 "object"。实际上，null 本身是基本类型。

#### 内置对象

JavaScript 中还有一些对象子类型，通常被称为内置对象。在 JavaScript中，它们实际上只是一些内置函数。这些内置函数可以当作构造函数来使用。
- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

JavaScript 中的函数是“一等公民”，因为它们本质上和普通的对象一样(只是可以调用)，所以可以像操作其他对象一样操作函数(比如当作另一个函数的参数)。

原始值 "I am a string" 并不是一个对象，它只是一个字面量，并且是一个不可变的值。 如果要在这个字面量上执行一些操作，比如获取长度、访问其中某个字符等，那需要将其转换为 String 对象。所幸，在必要时语言会自动把字符串字面量转换成一个 String 对象，如`strPrimitive.length`时，也就是说你并不需要显式创建一个对象。JavaScript 社区中的大多数人都认为能使用文字形式时就不要使用构造形式。同样的事也会发生在数值字面量上，如42.359.toFixed(2)。

null 和 undefined 没有对应的构造形式，它们只有文字形式。相反，Date 只有构造，没有文字形式。

对于 Object、Array、Function 和 RegExp(正则表达式)来说，无论使用文字形式还是构造形式，它们都是对象，不是字面量。在某些情况下，相比用文字形式创建对象，构造形式可以提供一些额外选项。由于这两种形式都可以创建对象，所以我们首选更简单的文字形式。建议只在需要那些额外选项时使用构造形式。

Error 对象很少在代码中显式创建，一般是在抛出异常时被自动创建。也可以使用 new Error(..) 这种构造形式来创建，不过一般来说用不着。

### 内容

引擎内部，对象值的存储方式是多种多样的，一般并不会存在对象容器内部。存储在对象容器内部的是这些属性的名称，它们就像指针一样，指向这些值真正的存储位置。

#### 属性名

.a 语法通常被称为“属性访问”，["a"] 语法通常被称为“键访问”。这两种语法的主要区别在于 . 操作符要求属性名满足标识符的命名规范，而 [".."] 语法可以接受任意 UTF-8/Unicode 字符串作为属性名。

**在对象中，属性名永远都是字符串。**如果你使用 string(字面量)以外的其他值作为属性名，那它首先会被转换为一个字符串。即使是数字也不例外。

**ES6** 增加了可计算属性名，可以在文字形式中使用 [] 包裹一个表达式来当作属性名，可计算属性名最常用的场景可能是 ES6 的符号(Symbol)。

#### 数组

你可以把数组当作一个普通的键 / 值对象来使用，并且不添加任何数值索引，但是这并不是一个好主意。数组和普通的对象都根据其对应的行为和用途进行了优化，所以最好只用对象来存储键 / 值对，只用数组来存储数值下标 / 值对。

**如果你试图向数组添加一个属性，但是属性名“看起来”像一个数字，那它会变成一个数值下标(因此会修改数组的内容而不是添加一个属性)**

```
var myArray = [ "foo", 42, "bar" ];
myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"

var myArray = [ "foo", 42, "bar" ];
myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

#### 复制对象

**复制的几个问题：**

- 对于深复制来说，如果属性间存在循环引用，这样就可能导致死循环。我们是应该检测循环引用并终止循环(不复制深层元素)?还是应当直接报错或者是选择其他方法?
- 我们还不确定“复制”一个函数意味着什么。有些人会通过 toString() 来序列化一个函数的源代码(但是结果取决于 JavaScript 的具体实现，而且不同的引擎对于不同类型的函数处理方式并不完全相同)。

对于 JSON 安全(可以被序列化为一个 JSON 字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象)的对象来说，有一种巧妙的复制方法:
`var newObj = JSON.parse( JSON.stringify( someObj ) );`。这种方法需要保证对象是 JSON 安全的，所以只适用于部分情况。

相比深复制，浅复制非常易懂并且问题要少得多，ES6 定义了 Object.assign(..) 方法来实现浅复制。Object.assign(..) 方法的第一个参数是目标对象，之后还可以跟一个或多个源对象。它会遍历一个或多个源对象的所有可枚举并把它们复制(使用 = 操作符赋值)到目标对象，最后返回目标对象。

#### 属性描述符

从 ES5 开始，所有的属性都具备了属性描述符。普通的对象属性对应的属性/数据描述符包含一个value，writable(可写)、 enumerable(可枚举)和 configurable(可配置)。

```
var myObject = {
	a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// value: 2,
// writable: true,
// enumerable: true,
// configurable: true
// }

```

##### 设置属性

在创建普通属性时属性描述符会使用默认值，我们也可以使用 Object.defineProperty(..) 来添加一个新属性或者修改一个已有属性(如果它是 configurable)并对特性进行设置。然而，一般来说你不会使用这种方式，除非你想修改属性描述符。

```
var myObject = {};
Object.defineProperty( myObject, "a", {
	  value: 2,
		writable: true,
		configurable: true,
		enumerable: true
	}
);
myObject.a; // 2
```

**Writable**

writable 决定是否可以修改属性的值。若为false，我们对于属性值的修改会静默失败(silently failed)。如果在严格模式下，这种方法会抛出TypeError错误。

简单来说，你可以把 writable:false 看作是属性不可改变，相当于你定义了一个空操作 setter。严格来说，如果要和 writable:false 一致的话，你的 setter 被调用时应当抛出一个 TypeError 错误。

**Configurable**

把 configurable 修改成 false 是单向操作，无法撤销。

只要属性是可配置的，就可以使用 defineProperty(..) 方法来修改属性描述符。不管是不是处于严格模式，尝试修改一个不可配置的属性描述符都会出错。一个小小的例外:即便属性是 configurable:false，我们还是可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。

除了无法修改，configurable:false 还会禁止删除这个属性。如果对象的某个属性是某个对象 / 函数的最后一个引用者，对这个属性执行 delete 操作之后，这个未引用的对象 / 函数就可以被垃圾回收。但是，不要把 delete 看作一个释放内存的工具(就像 C/C++ 中那样)，它就是一个删除对象属性的操作，仅此而已。

**Enumerable**

这个描述符控制的是属性是否会出现在对象的属性枚举中，比如说 for..in 循环。如果把 enumerable 设置成 false，这个属性就不会出现在枚举中，但仍然可以正常访问它。

用户定义的所有的普通属性默认都是 enumerable。

#### 不变性

有时候你会希望属性或者对象是不可改变(无论有意还是无意)。

##### 所有的方法创建的都是浅不变性

```
myImmutableObject.foo; // [1,2,3]
myImmutableObject.foo.push( 4 );
myImmutableObject.foo; // [1,2,3,4]
假设代码中的 myImmutableObject 已经被创建而且是不可变的，但是为了保护它的内容 myImmutableObject.foo，你还需要让 foo 也不可变。
```
##### 对象不变的方法

**对象常量**

结合 writable:false 和 configurable:false 就可以创建一个真正的常量属性(不可修改、 重定义或者删除)。

**禁止扩展**

若想禁止一个对象添加新属性并且保留已有属性，可以使用 Object.prevent Extensions(..)。

**密封**

Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。
所以，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性(虽然可以修改属性的值)。

**冻结**

Object.freeze(..) 会创建一个冻结对象，这个方法会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable:false，这样就无法修改它们的值。

你可以“深度冻结”一个对象，具体方法为，首先在这个对象上调用 Object.freeze(..)， 然后遍历它引用的所有对象并在这些对象上调用 Object.freeze(..)。但是一定要小心，因为这样做有可能会在无意中冻结其他(共享)对象。

#### Get与Put

语言规范中，myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作(有点像函数调用:[[Get]]())。1、首先在对象中查找是否有名称相同的属性， 如果找到就会返回这个属性的值。2、如果没有找到名称相同的属性，会在原型链上查找。3、如果始终都没有找到名称相同的属性，[[Get]] 操作会返回值 undefined。

[[Put]] 被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性(这是最重要的因素)。

如果已经存在这个属性，[[Put]] 算法大致会检查下面这些内容。

1. 属性是否是访问描述符？如果是并且存在setter就调用setter。
2. 属性的数据描述符中writable是否是false？如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

#### getter与setter

当你给一个属性定义 getter、setter 或者两者都有时，这个属性会被定义为“访问描述符”。对访问描述符，JavaScript 会忽略它们的 value 和 writable 特性，取而代之的是关心 set 和 get(还有 configurable 和 enumerable)特性。

```
var myObject = {
	// 给 a 定义一个 getter
	get a() {
		return 2;
	}
};
Object.defineProperty(
	myObject, // 目标对象
	"b", // 属性名
	{
		// 描述符
		// 给 b 设置一个 getter
		get: function(){ return this.a * 2 },
		// 确保 b 会出现在对象的属性列表中
    enumerable: true
  }
);
myObject.a; // 2
myObject.b; // 4
```


上述两者getter定义方法，都会在对象中创建一个不包含值的属性，对这个属性的访问会自动调用隐藏函数。

通常来说getter和setter是成对出现的。

#### 判断属性存在

如 myObject.a 的属性访问返回值可能是 undefined，但是这个值有可能是属性中存储的 undefined，也可能是因为属性不存在所以返回 undefined。那么如何区分这两种情况呢?

**判断对象是否具有某属性**

in 操作符会检查属性是否在对象及其 [[Prototype]] **原型链**中。hasOwnProperty(..) 只会检查属性是否在 myObject 对象中。

```
var myObject = {
	a:2
};
("a" in myObject); // true
("b" in myObject); // false
myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

所有的普通对象都可以通过对于 Object.prototype 的委托来访问 hasOwnProperty(..)， 但是有的对象可能没有连接到Object.prototype, 如通过Object.create(null)。在这种情况下，形如 myObejct.hasOwnProperty(..) 就会失败。这时可以通过Object.prototype.hasOwnProperty.call(myObject,"a")进行判断。


##### 枚举

“可枚举”就相当于“可以出现在对象属性的遍历中”。在数组上应用 for..in 循环有时会产生出人意料的结果，因为这种枚举不仅会包含所有数值索引，还会**包含所有可枚举属性(包含原型链)**。最好只在对象上应用 for..in 循环，如果要遍历数组就使用传统的 for 循环来遍历数值索引。

**判断是否可枚举**

propertyIsEnumerable(..) 会检查给定的属性名是否直接存在于对象中(而不是在原型链上)并且满足 enumerable:true。

**获取对象属性列表**

Object.keys(..) 会返回一个数组，包含所有可枚举属性，Object.getOwnPropertyNames(..) 会返回一个数组，包含所有属性，无论它们是否可枚举。它们只获取对象直接属性。

#### 遍历

##### 几个函数

- forEach(..) 会遍历数组中的所有值并忽略回调函数的返回值。
- every(..) 会一直运行直到对象回调函数返回 false(或者“假”值)。
- some(..) 会一直运行直到回调函数返回 true(或者 “真”值)。

every(..) 和 some(..) 中特殊的返回值和普通 for 循环中的 break 语句类似，它们会提前终止遍历。

##### 对象遍历

遍历对象属性时的顺序是不确定的，在不同的 JavaScript 引擎中可能不一样。因此，在不同的环境中需要保证一致性时，一定不要相信任何观察到的顺序，它们是不可靠的。

##### for..of 循环

ES6 增加了一种用来遍历数组的 for..of 循环语法。for..of 会寻找内置或者自定义的 @@iterator 对象并调用它的 next() 方法来遍历数据值。

```
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();
it.next();
// { value:1, done:false } it.next();
// { value:2, done:false } it.next();
// { value:3, done:false } it.next();
// { done:true }
```

和数组不同，普通的对象没有内置的 @@iterator，所以无法自动完成 for..of 遍历。你可以给任何想遍历的对象定义 @@iterator。

```
var myObject = {
	a: 2,
	b: 3
};
Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
          value: o[ks[idx++]],
          done: (idx > ks.length)
        };
			}
		};
	}
});
// 手动遍历 myObject
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }
```

## 类

### 什么是类

类 / 继承描述了一种代码的组织结构形式——一种在软件中对真实世界中问题领域的建模方法。好的设计就是把数据以及和它相关的行为打包(或者说封装)起来。我们还可以使用类对数据结构进行分类。

类理论强烈建议父类和子类使用相同的方法名来表示特定的行为，从而让子类重写父类。 而在 JavaScript 代码中这样做会降低代码的可读性和健壮性。其他语言中的类和 JavaScript 中的“类”并不一样，JavaScript 并不会(像类那样)自动创建对象的副本。

### 几个概念

- 构造函数。类实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为构造函数。
- 实例化。传统的类被实例化时，它的行为会被复制到实例中。
- 继承。在面向类的语言中，你可以先定义一个类，然后定义一个继承前者的类，后者通常被称为“子类”，前者通常被称为“父类”。类被继承时，行为会被复制到子类中。
- 多态。父类的通用行为可以被子类用更特殊的行为重写。相对多态性允许我们从重写行为中引用基础行为。多态看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果。

### 多重继承问题

钻石问题的变种：在钻石问题中，子类 D 继承自两个父类(B 和 C)，这两个父类都继承自 A。如果 A 中有 drive() 方法并且 B 和 C 都重写了这个方法 (多态)，那当 D 引用 drive() 时应当选择哪个版本呢(B:drive() 还是 C:drive())?

JavaScript 本身并不提供“多重继承”功能。

### 混入

在继承或者实例化时，JavaScript 的对象机制并不会自动执行复制行为，它们会被关联起来。

混入是Javascript中模拟类的复制行为的一种方法，分为显示混入和隐式混入。但是通常会产生丑陋并且脆弱的语法，比如显式伪多态(OtherObj.methodName.call(this, ...))，这会让代码更加难懂并且难以维护。

显式混入实际上无法完全模拟类的复制行为，因为对象(和函数!别忘了函数也是对象)只能复制引用，无法复制被引用的对象或者函数本身。


## 原型

### [[Prototype]]

#### 什么是[[Prototype]]

JavaScript 中的对象有一个特殊的 **[[Prototype]] 内置属性**，是对于其他对象的引用。几乎所有的对象在创建时 [[Prototype]] 属性都会被赋予一个非空的值。

当你试图引用对象的属性时会触发[[Get]] 操作，对于默认的 [[Get]] 操作，如果无法在对象本身找到需要的属性，就会继续访问对象的 [[Prototype]] 链。

Object.create(..) 能创建一个对象并把这个对象的 [[Prototype]] 关联到指定的对象。

所有普通的 [[Prototype]] 链最终都会指向内置的 Object.prototype。

#### [[Prototype]]应用-属性设置与屏蔽

给一个对象设置属性如myObject.foo = "bar"的过程:

1. 若myObject 对象中包含名为 foo 的普通数据访问属性，这条赋值语句只会修改已有的属性值。
2. 若 foo 不是直接存在于 myObject 中，[[Prototype]] 链就会被遍历，类似 [[Get]] 操作。如果原型链上找不到 foo，foo 就会被直接添加到 myObject 上。
3. 若原型链上能找到 foo
	1. 如果在[[Prototype]]链上层存在名为foo的普通数据访问属性并且没有被标记为只读(writable:false)，那就会直接在 myObject 中添加一个名为 foo 的新属性，它是屏蔽属性。
	2. 如果在[[Prototype]]链上层存在foo，但是它被标记为只读(writable:false)，那么无法修改已有属性或者在 myObject 上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
		- 只读属性会阻止 [[Prototype]] 链下层隐式创建(屏蔽)同名属性。这样做主要是为了模拟类属性的继承。你可以把原型链上层的 foo 看作是父类中的属性，它会被 myObject 继承(复制)，这样一来 myObject 中的 foo 属性也是只读，所以无法创建。
	3. 如果在[[Prototype]]链上层存在foo并且它是一个setter，那就一定会调用这个 setter。foo 不会被添加到(或者说屏蔽于)myObject，也不会重新定义 foo 这个 setter。

```
anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false
myObject.a++; // 隐式屏蔽! anotherObject.a; // 2
myObject.a; // 3
myObject.hasOwnProperty( "a" ); // true
```

### Javascript中的“类”

在Javascript中，“委托”相比类是一个更合适的术语，因为对象之间的关系不是复制而是委托。

#### prototype来自哪

所有的**函数**默认都会拥有一个名为 prototype 的公有并且不可枚举的属性，它会指向另一个对象。

```
function Foo() {
	// ...
}
var a = new Foo();
```
prototype 是在调用 new Foo()时创建的，最后会被关联到这个“Foo.prototype”对象上。new Foo() 还会创建 a，其中的一步就是给 a 一个内部的 [[Prototype]] 链接，关联到 Foo.prototype 指向的那个对象。最后我们得到了两个对象，它们之间互相关联。

#### constructor

Foo.prototype 默认(在代码中第一行声明时)有一个公有并且不可枚举的属性 .constructor，这个属性引用的是对象关联的函数(本例中是 Foo)。 a 没有 .constructor 属性，虽然 a.constructor 指向 Foo 函数，但是这个 constructor 是来自 Foo 的 prototype。如果你创建了一个新对象并替换了函数默认的 .prototype 对象引用，那么新对象并不会自动获得 .constructor 属性。

```
function Foo() { /* .. */ }
Foo.prototype = { /* .. */ }; // 创建一个新原型对象
var a1 = new Foo();
a1.constructor === Foo; // false! a1.constructor === Object; // true!
```

.constructor 并不是一个不可变属性。它是不可枚举的，但是它的值是可以被修改的。此外，你可以给任意 [[Prototype]] 链中的任意对象添加一个名为 constructor 的属性或者对其进行修改，你可以任意对其赋值。因此，一些随意的对象属性引用，比如 a1.constructor，实际上是不被信任的，它们不一定会指向默认的函数引用。
**a1.constructor 是一个非常不可靠并且不安全的引用**。通常来说要尽量避免使用这些引用。

#### 原型继承

Object.create(..) 会凭空创建一个“新”对象并把新对象内部的 [[Prototype]] 关联到你指定的对象(本例中是 Foo.prototype)。

如果我们想创建一个新对象并把它关联到我们希望的对象上`Bar.prototype = new Foo();`基本上满足你的需求，但是可能会产生一些副作用。如果函数 Foo 有一些副作用(比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等)的话，就会影响到 Bar() 的“后代”，后果不堪设想。

因此，要创建一个合适的关联对象，我们必须使用 Object.create(..) 而不是使用具有副作用的 Foo(..)。这样做唯一的缺点就是需要创建一个新对象然后把旧对象(prototype)抛弃掉。

如果能有一个标准并且可靠的方法来修改对象的 **[[Prototype]]** 关联就好了。
- 在 ES6 之前， 我们只能通过设置 .__proto__ 属性来实现，但是这个方法并不是标准并且无法兼容所有浏览器。
- ES6 添加了辅助函数 Object.setPrototypeOf(..)，可以用标准并且可靠的方法来修改关联。

```
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );

// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

如果忽略掉 Object.create(..) 方法带来的轻微性能损失(抛弃的对象需要进行垃圾回收)，它实际上比 ES6 及其之后的方法更短而且可读性更高。不过无论如何，这是两种完全不同的语法。

#### 检查“类”的关系

检查一个实例(JavaScript 中的对象)的继承祖先(JavaScript 中的委托关联)通常被称为内省(或者反射)。

**通过内省找出 a 的“祖先”(委托关联)**

- instanceof。a instanceof Foo 回答的问题是，在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象。这种方法只能判断对象与构造函数之间的关系，如果你想判断两个对象(比如 a 和 b)之间是否通过 [[Prototype]] 链关联，只用 instanceof 无法实现。
- isPrototypeOf(..) 。Foo.prototype.isPrototypeOf( a ) 回答的问题是，在 a 的整条 [[Prototype]] 链中是否出现过 Foo.prototype。
- 在 ES5 中可以直接获取一个对象的 [[Prototype]] 链。`Object.getPrototypeOf( a ) === Foo.prototype; // true`。

#### \_\_prototype\_\_

和 .constructor 一样，.\_\_proto\_\_ 并不存在于你正在使用的对象中。它存在于内置的 Object.prototype。 此外，.\_\_proto\_\_ 看起来很像一个属性，但是实际上它更像一个 getter/setter。访问(获取值)a.\_\_proto\_\_ 时，实际上是调用了 a.\_\_proto\_\_()(调用 getter 函数)。

.__proto__ 的实现大致上是这样:

```
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// ES6 中的 setPrototypeOf(..)
		Object.setPrototypeOf( this, o );
		return o;
	}
});
```

.\_\_proto\_\_ 是可设置属性，之前的代码中使用 ES6 的 Object.setPrototypeOf(..) 进行设置。然而，通常来说你不需要修改已有对象的 [[Prototype]]，最好把 [[Prototype]] 对象关联看作是只读特性，从而增加代码的可读性。

#### 对象关联

##### 创建关联

我们并不需要类来创建两个对象之间的关系，只需要通过委托来关联对象就足够了。而 Object.create(..) 不包含任何“类的诡计”，所以它可以完美地创建我们想要的关联关系。

Object.create(..) 会创建一个新对象(bar)并把它关联到我们指定的对象(foo)，这样我们就可以充分发挥 [[Prototype]] 机制的威力(委托)并且避免不必要的麻烦(比如使用 new 的构造函数调用会生成 .prototype 和 .constructor 引用)。

Object.create(null) 会创建一个拥有空(或者说 null)[[Prototype]] 链接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以 instanceof 操作符(之前解释过)无法进行判断，因此总是会返回 false。 这些特殊的空 [[Prototype]] 对象通常被称作“字典”，它们完全不会受到原型链的干扰，因此非常适合用来存储数据。

**Object.create()的polyfill代码**

```
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

标准 ES5 中内置的 Object.create(..) 函数还提供了一系列附加功能，Object.create(..) 的第二个参数指定了需要添加到新对象中的属性名以及这些属性的属性描述符。因为 ES5 之前的版本无法模拟属性操作符，所以 polyfill 代码无法实现这个附加功能。

#### 利用关联作为备用

看起来对象之间的关联关系是处理“缺失”属性或者方法时的一种备用选项。如果你只是将关联关系作为备用，那么你的软件就会变得有点“神奇”，而且很难理解和维护。

在 ES6 中有一个被称为“代理”(Proxy)的高端功能，它实现的就是“方法无法找到”时的行为。

## 行为委托

JavaScript 中[[Prototype]] 机制的本质就是对象之间的关联关系。委托行为意味着某些对象在找不到属性或者方法引用时会把这个请求委托给另一
个对象(Task)。

### 面向委托的设计

这个设计模式要求尽量少使用容易被重写的通用方法名，提倡使用更有描述性的方法名，尤其是要写清相应对象行为的类型。这样做实际上可以创建出更容易理解和维护的代码，因为方法名(不仅在定义的位置，而是贯穿整个代码)更加清晰(自文档)。

在委托设计模式中，除了建议使用不相同并且更具描述性的方法名之外，还要通过对象关联避免丑陋的显式伪多态调用(Widget.call 和 Widget.prototype.render.call)，代之以简单的相对委托调用 this.init(..) 和 this.insert(..)。

你无法在两个或两个以上互相(双向)委托的对象之间创建循环委托。如果你把 B 关联到 A 然后试着把 A 关联到 B，就会出错。

JavaScript中的函数之所以可以访问 call(..)、apply(..) 和 bind(..)，就是因为**函数本身是对象**。而函数对象同样有 [[Prototype]] 属性并且关联到 **Function.prototype** 对象。

在委托设计模式中，除了建议使用不相同并且更具描述性的方法名之外，还要通过对象关联避免丑陋的显式伪多态调用(Widget.call 和 Widget.prototype.render.call)，代之以简单的相对委托调用 this.init(..) 和 this.insert(..)。

#### 示例

```
var Widget = {
	init: function(width,height){
		this.width = width || 50; this.height = height || 50; this.$elem = null;
	},
	insert: function($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px" } ).appendTo( $where );
		}
	}
};

var Button = Object.create( Widget );
Button.setup = function(width,height,label){
	// 委托调用
	this.init( width, height );
	this.label = label || "Default";
	this.$elem = $( "<button>" ).text( this.label );
};
Button.build = function($where) {
	// 委托调用
	this.insert( $where );
	this.$elem.click( this.onClick.bind( this ) );
};
```

之前的一次调用(var btn1 = new Button(..))现在变成了两次(var btn1 = Object.create(Button) 和 btn1.setup(..))。乍一看这似乎是一个缺点 (需要更多代码)。但是这一点其实也是对象关联风格代码相比传统原型风格代码有优势的地方。使用类构造函数的话，你需要(并不是硬性要求，但是强烈建议)在同一个步骤中实现构造和初始化。然而，在许多情况下把这两步分开(就像对象关联代码一样)更灵活。

举例来说，假如你在程序启动时创建了一个实例池，然后一直等到实例被取出并使用时才执行特定的初始化过程。这个过程中两个函数调用是挨着的，但是完全可以根据需要让它们出现在不同的位置。对象关联可以更好地支持关注分离(separation of concerns)原则，创建和初始化并不需要合并为一个步骤。

### ES6的class语法糖

class 很好地伪装成 JavaScript 中类和继承设计模式的解决方案，但是它实际上起到了反作用:它隐藏了许多问题并且带来了更多更细小但是危险的问题。class 加深了过去 20 年中对于 JavaScript 中“类”的误解，在某些方面，它产生的问题比解决的多，而且让本来优雅简洁的 [[Prototype]] 机制变得非常别扭。

```
class Widget {
	constructor(width,height) {
		this.width = width || 50;
		this.height = height || 50;
		this.$elem = null;
  }
  render($where){
		if (this.$elem) {
			this.$elem.css( {
				width: this.width + "px",
				height: this.height + "px"
			} ).appendTo( $where );
		}
	}
}
class Button extends Widget {
	constructor(width,height,label) {
		super( width, height );
		this.label = label || "Default";
		this.$elem = $( "<button>" ).text( this.label );
  }
  render($where) {
		super( $where );
		this.$elem.click( this.onClick.bind( this ) );
	}
  onClick(evt) {
     console.log( "Button '" + this.label + "' clicked!" );
	}
}
```

使用 ES6 的 class 之后，许多丑陋的语法都不见了，尽管语法上得到了改进，但实际上这里并没有真正的类，class 仍然是通过 [[Prototype]] 机制实现的。

除了语法更好看之外，ES6 还解决了什么问题?

1. 不再引用杂乱的 .prototype 了。
2. Button声明时直接“继承”了Widget，不再需要通过Object.create(..)来替换 .prototype 对象，也不需要设置 .\_\_proto\_\_ 或者 Object.setPrototypeOf(..)。
3. 可以通过super(..)来实现相对多态，这样任何方法都可以引用原型链上层的同名方法。这可以解决第 4 章提到过的那个问题:构造函数不属于类，所以无法互相引用——
super() 可以完美解决构造函数的问题。
4. class 字面语法**不能声明属性**(只能声明方法)。看起来这是一种限制，但是它会排除掉许多不好的情况，如果没有这种限制的话，原型链末端的“实例”可能会意外地获取其他地方的属性(这些属性隐式被所有“实例”所“共享”)。所以，class 语法实际上可以帮助你避免犯错。
5. 可以通过extends很自然地扩展对象(子)类型，甚至是内置的对象(子)类型，比如 Array 或 RegExp。没有 class ..extends 语法时，想实现这一点是非常困难的，基本上只有框架的作者才能搞清楚这一点。但是现在可以轻而易举地做到!


然而，class 语法并没有解决所有的问题，在 JavaScript 中使用“类”设计模式仍然存在许多深层问题：

1. class 基本上只是现有 [[Prototype]](委托)机制的一种语法糖。也就是说，class 并不会像传统面向类的语言一样在声明时静态复制所有行为。如果你 (有意或无意)修改或者替换了父“类”中的一个方法，那子“类”和所有实例都会受到影响，因为它们只是使用基于 [[Prototype]] 的实时委托:
2. ES6 中的 class 语法会让传统类和委托对象之间的区别更加难以发现和理解。
3. class 语法无法定义类成员属性(只能定义方法)，如果为了跟踪实例之间共享状态必须要这么做，那你只能使用丑陋的 .prototype 语法，像这样:

	```
	class C {
	    constructor() {
	        // 确保修改的是共享状态而不是在实例上创建一个屏蔽属性!
	        C.prototype.count++;
	        // this.count 可以通过委托实现我们想要的功能
	        console.log("Hello: " + this.count);
	    }
	}
	// 直接向 prototype 对象上添加一个共享状态 C.prototype.count = 0;
	var c1 = new C(); // Hello: 1
	var c2 = new C(); // Hello: 2
	c1.count === 2; // true
	c1.count === c2.count; // true
	```
这种方法最大的问题是，它违背了 class 语法的本意，在实现中暴露了 .prototype。

4. class 语法面临意外屏蔽的问题。

	```
	class C {
	    constructor(id) {
	        // 噢，郁闷，我们的 id 属性屏蔽了 id() 方法
	        this.id = id;
	    }
	    id() {
	        console.log("Id: " + id);
	    }
	}
	var c1 = new C("c1");
	c1.id(); // TypeError -- c1.id 现在是字符串 "c1"
	```
5. super 也存在一些非常细微的问题。出于性能考虑(this 绑定已经是很大的开销了)，super 并不是动态绑定的，它会在声明时“静态”绑定。如果你和大多数 JavaScript 开发者一样，会用许多不同的方法把函数应用在不同的(使用 class 定义的)对象上，那你可能不知道，每次执行这些操作时都必须重新绑定 super。此外，根据应用方式的不同，super 可能不会绑定到合适的对象。因此，对于引擎自动绑定的 super 来说，你必须时刻警惕是否需要进行手动绑定。

### 更简洁的设计

对象关联除了能让代码看起来更简洁(并且更具扩展性)外还可以通过行为委托模式简化代码结构。我们来看最后一个例子，它展示了对象关联如何简化整体设计。

```
var LoginController = {
    errors: [],
    getUser: function() {
        return document.getElementById("login_username").value;
    },
    getPassword: function() {
        return document.getElementById("login_password").value;
    },
    validateEntry: function(user, pw) {
        user = user || this.getUser();
        pw = pw || this.getPassword();
        if (!(user && pw)) {
            return this.failure(
                "Please enter a username & password!"
            );
        } else if (user.length < 5) {
            return this.failure(
                "Password must be 5+ characters!"
            );
        }
        // 如果执行到这里说明通过验证
        return true;
    },
    showDialog: function(title, msg) { // 给用户显示标题和消息
    },
    failure: function(err) {
        this.errors.push(err);
        this.showDialog("Error", "Login invalid: " + err);
    }
};
// 让 AuthController 委托 LoginController
var AuthController = Object.create(LoginController);
AuthController.errors = [];
AuthController.checkAuth = function() {
    var user = this.getUser();
    var pw = this.getPassword();
    if (this.validateEntry(user, pw)) {
        this.server("/check-auth", {
                user: user,
                pw: pw
            })
            .then(this.accepted.bind(this))
            .fail(this.rejected.bind(this));
    }
};
AuthController.server = function(url, data) {
    return $.ajax({
        url: url,
        data: data
    });
};
AuthController.accepted = function() {
    this.showDialog("Success", "Authenticated!")
};
AuthController.rejected = function(err) {
    this.failure("Auth Failed: " + err);
};
```

### 更好的语法

在 ES6 中我们可以在任意对象的字面形式中使用简洁方法声明，可以让对象关联风格更加人性化(并且仍然比典型的原型风格代码更加简洁和优秀)。你完全不需要使用类就能享受整洁的对象语法。

```
// 使用更好的对象字面形式语法和简洁方法
var AuthController = {
    errors: [],
    checkAuth() {
        // ...
    },
    server(url, data) {
        // ...
    }
    // ...
};

// 现在把 AuthController 关联到 LoginController
Object.setPrototypeOf( AuthController, LoginController );
```

简洁方法有一个非常小但是非常重要的**缺点**。

```
var Foo = {
	bar() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```
去掉语法糖之后的代码如下所示:

```
var Foo = {
	bar: function() { /*..*/ },
	baz: function baz() { /*..*/ }
};
```
由于函数对象本身没有名称标识符，所以bar()的缩写形式(function()..)实际上会变成一个匿名函数表达式并赋值给 bar 属性。

下面是匿名函数（没有 name 标识符）表达式的三大主要缺点：
1. 调试栈更难追踪;
2. 自我引用(递归、事件(解除)绑定，等等)更难;
3. 代码(稍微)更难理解。

简洁方法没有第 1 和第 3 个缺点。去掉语法糖的版本使用的是匿名函数表达式，通常来说并不会在追踪栈中添加 name，但是简洁方法会给对应的函数对象设置一个内部的 name 属性，这样理论上可以用在追踪栈中。(但是追踪的具体实现是不同的，因此无法保证可以使用。)

简洁方法无法避免第 2 个缺点，它们不具备可以自我引用的词法标识符。

```
var Foo = {
    bar: function(x) {
        if (x < 10) {
            return Foo.bar(x * 2);
        }
        return x;
    },
    baz: function baz(x) {
        if (x < 10) {
            return baz(x * 2);
        }
        return x;
    }
};
```

使用简洁方法时一定要小心这一点。如果你需要自我引用的话，那最好使用传统的具名函数表达式来定义对应的函数( · baz: function baz(){..}· )，不要使用简洁方法。

ES6 的 isPrototypeOf(..)、getPrototypeOf(..)内省方法相比 instanceof 更加简洁并且清晰。

### 小结

在软件架构中你可以选择是否使用类和继承设计模式。除了类设计模式，还有另一种更少见但是更强大的设计模式:行为委托。

行为委托认为对象之间是兄弟关系，互相委托，而不是父类和子类的关系。JavaScript 的 [[Prototype]] 机制本质上就是行为委托机制。也就是说，我们可以选择在 JavaScript 中努力实现类机制，也可以拥抱更自然的 [[Prototype]] 委托机制。

当你只用对象来设计代码时，不仅可以让语法更加简洁，而且可以让代码结构更加清晰。 对象关联(对象之前互相关联)是一种编码风格，它倡导的是直接创建和关联对象，不把
它们抽象成类。对象关联可以用基于 [[Prototype]] 的行为委托非常自然地实现。

## 资料

* [You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20&%20object%20prototypes/README.md#you-dont-know-js-this--object-prototypes)