# 类型和语法

## 类型

### 什么是类型

对语言引擎和开发人员来说，类型是值的内部特征，它定义了值的行为，以使其区别于其他值。如果语言引擎和开发人员对42（数字）和"42"（字符串）采取不同的处理方式，那就说明它们是不同的类型。

### 为什么要懂类型

要正确合理地进行类型转换，我们必须掌握JavaScript中的各个类型及其内在行为。几乎所有的JavaScript程序都会涉及某种形式的强制类型转换，处理这些情况时我们需要有充分的把握和自信。

### 内置类型

- null
- undefined
- boolean
- number
- string
- object
- symbol -- ES6新增!

除了Object，其他统称为“基本类型/原始类型”。

可以使用typeof来查看值的类型，它总会返回一个字符串。有两种特殊情况`typeof null === 'object'`-JavaScript的bug、`typeof function(){} === 'function'`。

已在作用域中声明但还没有赋值的变量，是undefined的。相反，还没有在作用域中声明过的变量，是undeclared的。对于undeclared（或者not defined）变量，typeof照样返回"undefined"。这是因为typeof有一个特殊的**安全防范机制**，该安全防范机制对在浏览器中运行的JavaScript代码来说是有帮助的，因为多个脚本文件会在共享的全局命名空间中加载变量，typeof使得在程序中**检查全局变量**不会出现ReferenceError错误，相反，直接if undeclared全局变量会报错。（还有一种不用通过typeof的安全防范机制的方法，就是检查所有全局变量是否是全局对象的属性，浏览器中的全局对象是window。）

function 是 object 的**子类型**，它是**可调用对象**，内部有一个[[Call]]属性使得其可被调用。函数对象的length属性是其声明的参数个数。

## 值

### 数组

#### 简介

在JavaScript中，数组可以容纳任何类型的值。数组也是对象，可以包含字符串键值和属性（但这些并不计算在数组长度内）。但在数组中加入字符串键值/属性并不是一个好主意，建议使用对象来存放键值/属性值，用数组来存放数字索引值。

**如果字符串键值能够被强制类型转换为十进制数字的话(a['12'])，它就会被当作数字索引来处理。**使用**delete运算符**可以将单元从数组中删除，单元删除后，数组的length属性并不会发生变化，数组也变成了一个**稀疏数组**(含有空缺单元的数组)。

#### 类数组

类数组是一组能通过数字索引的值，但不是真正的数组。例如，一些DOM查询操作会返回DOM元素列表，它们并非真正意义上的数组，arguments对象也说类数组（从ES6开始已废止）。
。

**有时我们需要将类数组转换成真正的数组。**slice(..)经常被用于这类转换，slice()会返回类数组的一个数组复本`Array.prototype.slice.call(arguments)`。ES6中的内置工具函数Array.from(..)也能实现同样的功能。

### 字符串

字符串和数组很相似，都有length属性以及indexOf(..)和concat(..)方法。不同之处在于JavaScript中字符串是不可变(字符串的成员函数不会改变其原始值，而是创建并返回一个新的字符串)的，而数组是可变(数组的成员函数都是在其原始值上进行操作)的。

许多数组函数用来处理字符串很方便。虽然字符串没有这些函数，但可以通过**借用**数组的非变更方法来处理字符串。不过我们无法“借用”数组的**可变更成员函数**(reverse..)，因为字符串是不可变的。一个变通的办法是先将字符串split转换为数组，待处理完后再将结果join回字符串，不过这个方法对于包含**复杂字符**（Unicode，如星号、多字节字符等）的字符串并不适用。这时则需要功能更加完备、能够处理Unicode的工具库。

### 数字

与其他语言不同，JavaScript没有真正意义上的整数。JavaScript中的“整数”就是没有小数的十进制数，所以42.0即等同于“整数”42。
与大部分现代编程语言（包括几乎所有的脚本语言）一样，JavaScript中的数字类型是基于IEEE 754标准来实现的，该标准通常也被称为**浮点数**。JavaScript使用的是“双精度”格式（即64位二进制）。

Javascript能够呈现的最大浮点数大约是1.798e+308（这是一个相当大的数字），它定义在**Number.MAX_VALUE**中。最小浮点数定义在**Number.MIN_VALUE**中，大约是5e-324，它不是负数，但无限接近于0。

Javascript中特别大和特别小的数字默认用**指数格式**显示，与toExponential()函数的输出结果相同。

数字值可以调用Number.prototype中的方法。tofixed(..)方法可指定小数部分的显示位数，如果指定的小数部分的显示位数多于实际位数就用0补齐。toPrecision(..)方法用来指定有效数位的显示位数。

对于**．运算符**需要给予特别注意，因为它是一个有效的数字字符，会被优先识别为数字字面量的一部分，然后才是对象属性访问运算符。42.toFixed(3)是无效语法，因为．被视为常量42．的一部分。

```
// 无效语法
42.toFixed( 3 );	// SyntaxError

// 下面的语法都有效
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	 // "42.000"
```

ES6使用0x、0b和0o前缀，分别表示十六进制、二进制、八进制。

#### 较小的数值

```
.1 + .2 === 0.30000000000000004 // true
```

二进制浮点数中的0.1和0.2并不是十分精确，它们相加的结果并非刚好等于0.3。在处理带有小数的数字时需要特别注意。很多程序只需要处理整数，最大不超过百万或者万亿，此时使用JavaScript的数字类型是绝对安全的。

**那么应该怎样来判断0.1 + 0.2和0.3是否相等呢？**

最常见的方法是设置一个误差范围值，通常称为“机器精度”（machine epsilon），通过判断两个数字差值是否在精度范围内来判断相等。对JavaScript的数字来说，这个值通常是2^-52 (2.220446049250313e-16)，从ES6开始，该值定义在**Number.EPSILON**中，我们可以直接拿来用。

#### 整数安全范围

数字的呈现方式决定了“整数”的安全值范围远远小于Number.MAX_VALUE。能够被“安全”呈现的最大整数是**Number.MAX_SAFE_INTEGER**===2^53-1，即9007199254740991。最小整数是**Number.MIN_SAFE_INTEGER**===-9007199254740991。

Number.MAX_VALUE表示可能精确表示的最大浮点数，Number.MAX_SAFE_INTEGER表示保证精确表示的最大整数。

#### 整数检测

ES6中提供了**Number.isInterger(...)**方法。

ES6之前的polyfill Number.isInteger(..)方法：

```
if (! Number.isInteger) {
  Number.isInteger = function(num) {
    return typeof num == "number" && num % 1 == 0;
  };
}
```

要检测一个值是否是安全的整数，可以使用ES6中的**Number.isSafeInteger(..)**方法。

#### 32位有符号整数

虽然整数最大能够达到53位，但是有些数字操作（如数位操作）只适用于32位数字，所以这些操作中数字的安全范围就要小很多，变成从Math.pow(-2,31)（-2147483648，约－21亿）到Math.pow(2,31) -1（2147483647，约21亿）。

a | 0可以将变量a中的数值转换为32位有符号整数，因为某些特殊的值并不是32位安全范围的，如NaN和Infinity，此时会对它们执行抽象操作（abstract operation）ToInt32，以便转换为符合数位运算符要求的+0值。

#### 特殊数字

##### NaN

NaN是一个“警戒值”（sentinel value，有特殊用途的常规值），用于指出数字类型中的错误情况，即“执行数学运算没有成功，这是失败后返回的结果”。它是是唯一一个不等于自身的值。

可以使用内建的全局工具函数isNaN(..)来判断一个值是否是NaN。
然而，isNaN(..)有一个严重的缺陷，它的检查方式过于死板，就是“检查参数是否不是NaN，也不是数字”-这个bug自JavaScript问世以来就一直存在，至今已超过19年。

从ES6开始我们可以使用工具函数**Number.isNaN(..)**。很多JavaScript程序都可能存在NaN方面的问题，所以我们应该尽量使用Number.isNaN(..)这样可靠的方法，无论是系统内置还是polyfill。

##### 无穷数

如果数学运算（如加法）的结果超出处理范围，则由IEEE 754规范中的“就近取整”（round-to-nearest）模式来决定最后的结果。例如，相对于Infinity, Number.MAX VALUE + Math.pow(2, 969)与Number.MAX VALUE更为接近，因此它被“向下取整”（round down）；而Number.MAX VALUE + Math.pow(2, 970)与Infinity更为接近，所以它被“向上取整”（round up）。

计算结果一旦溢出为无穷数（infinity）就无法再得到有穷数。从数学运算和JavaScript语言的角度来说，Infinity/Infinity是一个未定义操作，结果为NaN。

##### 零值

根据规范，对负零进行字符串化会返回"0"。如果反过来将其从字符串转换为数字，得到的结果是准确的`+"-0" -> -0`。

`0 === -0`。要区分-0和0，还需要做一些特殊处理。

```
function isNegZero(n) {
	n = Number( n );
	return (n === 0) && (1 / n === -Infinity);
}￼
```

**我们为什么需要负零呢？**

有些应用程序中的数据需要以级数形式来表示（比如动画帧的移动速度），数字的符号位（sign）用来代表其他信息（比如移动的方向）。此时如果一个值为0的变量失去了它的符号位，它的方向信息就会丢失。所以保留0值的符号位可以防止这类情况发生。

##### 特殊等式

NaN和-0在相等比较时的表现有些特别，ES6中新加入了一个工具方法Object.is(..)来判断两个值是否绝对相等。

能使用==和===就尽量不要使用Object.is(..)，因为前者效率更高、更为通用。Object.is(..)主要用来处理那些特殊的相等比较。

### null与undefined

undefined类型只有一个值，即undefined。null类型也只有一个值，即null。它们的名称既是类型也是值。

#### null

null指空值（empty value）。

null是一个特殊关键字，不是标识符，我们不能将其当作变量来使用和赋值。

#### undefined

undefined指没有值（missing value）。

然而undefined却是一个标识符，可以被当作变量来使用和赋值。非严格模式下，我们可以为全局标识符undefined赋值（这样的设计实在是欠考虑！）。在非严格和严格两种模式下，我们可以声明一个名为undefined的局部变量。再次强调最好不要这样做！

表达式void______没有返回值，因此返回结果是undefined。void并不改变表达式的结果，只是让表达式不返回值。按惯例我们用void 0来获得undefined。void 0、void 1和undefined之间并没有实质上的区别。void运算符在其他地方也能派上用场，比如不让表达式返回任何结果（即使其有副作用）。总之，如果要将代码中的值（如表达式的返回值）设为undefined，就可以使用void。这种做法并不多见，但在某些情况下却很有用。

### 值和引用

简单值（即标量基本类型值，scalar primitive）总是通过值复制的方式来赋值/传递，包括null、undefined、字符串、数字、布尔和ES6中的symbol。复合值（compound value）——对象和函数，则总是通过引用复制的方式来赋值/传递。

**`arr.length = 0`**可以清空一个数组里面的数据。slice(..)不带参数会返回当前数组的一个浅复本（shallow copy）。

## 原生函数

原生函数可以被当作构造函数来使用，但其构造出来的对象可能会和我们设想的有所出入：`var a = new String( "abc" ); typeof a;// 是"object"，不是"String"￼`。

### [[Class]]

所有typeof返回值为"object"的对象（如数组）都包含一个内部属性[[Class]]。这个属性无法直接访问，一般通过Object.prototype.toString(..)来查看。多数情况下，对象的内部[[Class]]属性和创建该对象的内建原生构造函数相对应（如下），但并非总是如此。虽然Null()和Undefined()这样的原生构造函数并不存在，但是内部[[Class]]属性值仍然是"Null"和"Undefined"。

```
Object.prototype.toString.call( null );
// "[object Null]"
Object.prototype.toString.call( undefined );
// "[object Undefined]"
```

Object.create(null)、Object.create(undefined)生成的是普通对象，`Object.create(null).toString() === "[object Object]"`。ES5到ES6, toString()和[[Class]]的行为发生了一些变化，

### 封装对象

由于基本类型值没有．length和．toString()这样的属性和方法，需要通过封装对象才能访问，此时JavaScript会自动为基本类型值包装（box或者wrap）一个封装对象。

浏览器已经为．length这样的常见情况做了性能优化，直接使用封装对象来“提前优化”代码反而会降低执行效率。最好的办法是让JavaScript引擎自己决定什么时候应该使用封装对象。换句话说，就是应该优先考虑使用"abc"和42这样的基本类型值，而非new String("abc")和new Number(42)。

如果想要自行封装基本类型值，可以使用**Object(..)**函数（不带new关键字），他会根据传入参数的类型自动封装成对应的对象。

### 拆封对象

如果想要得到封装对象中的基本类型值，可以使用valueOf()函数。在需要用到封装对象中的基本类型值的地方会发生隐式拆封。

### 原生函数作为构造函数

应该尽量避免使用构造函数创建对象，除非十分必要，因为它们经常会产生意想不到的结果。

#### Array(..)

数组并没有预设长度这个概念。Array(..)创建出来的只是一个空数组，只不过它的length属性被设置成了指定的值。

构造函数Array(..)不要求必须带new关键字。不带时，它会被自动补上。因此Array(1,2,3)和new Array(1,2,3)的效果是一样的。Array构造函数只带**一个数字参数**的时候，该参数会被作为数组的预设长度（length），而非只充当数组中的一个元素。这实非明智之举：一是容易忘记，二是容易出错。

如若一个数组没有任何单元，但它的length属性中却显示有单元数量，这样奇特的数据结构会导致一些怪异的行为。

```
a.join( "-" ); // "--"
b.join( "-" ); // "--"
a.map(function(v, i){ return i; }); // [ undefined x 3 ]
b.map(function(v, i){ return i; }); // [ 0, 1, 2 ]
```
join(..)会首先假定数组不为空，然后通过length属性值来遍历其中的元素。而map(..)并不做这样的假定，因此结果也往往在预期之外，并可能导致失败。

我们将包含至少一个“空单元”的数组称为“稀疏数组”。只要将length属性设置为超过实际单元数的值，就能隐式地制造出空单元。另外还可以通过delete b[1]在数组b中制造出一个空单元。

虽然Array.apply( null, { length: 3 } )等效于Array(undefined, undefined, undefined)，其结果远比Array(3)更准确可靠。总之，**永远不要创建和使用空单元数组**。

#### Object(..)、Function(..)和RegExp(..)

除非万不得已，否则尽量不要使用Object(..)/Function(..)/RegExp(..)。

实际情况中没有必要使用new Object()来创建对象，因为这样就无法像常量形式那样一次设定多个属性，而必须逐一设定。构造函数Function只在极少数情况下很有用，比如动态定义函数参数和函数体。强烈建议使用常量形式（如/^a＊b+/g）来定义正则表达式，这样不仅语法简单，执行效率也更高，因为JavaScript引擎在代码执行前会对它们进行预编译和缓存。

RegExp(..)有时还是很有用的，比如动态定义正则表达式时。


#### Date(..)和Error(..)

相较于其他原生构造函数，Date(..)和Error(..)的用处要大很多，因为没有对应的常量形式来作为它们的替代。

Date(..)主要用来获得当前的Unix时间戳（从1970年1月1日开始计算，以秒为单位）。该值可以通过日期对象中的getTime()来获得。从ES5开始引入了一个更简单的方法，即静态函数Date.now()。

构造函数Error(..)（与前面的Array()类似）带不带new关键字都可。创建错误对象（error object）主要是为了获得当前运行栈的上下文（大部分JavaScript引擎通过只读属性．stack来访问）。栈上下文信息包括函数调用栈信息和产生错误的代码行号，以便于调试（debug）。

还有一些针对特定错误类型的原生构造函数，这些构造函数很少被直接使用，它们在程序发生异常（比如试图使用未声明的变量产生ReferenceError错误）时会被自动调用。

#### Symbol(..)

ES6中新加入了一个基本数据类型——符号（Symbol）。符号并非对象，而是一种简单标量基本类型。我们可以使用Symbol(..)原生构造函数来自定义符号。但它比较特殊，不能带new关键字，否则会出错。

符号具有唯一性的特殊值（并非绝对），用它来命名对象属性不容易导致重名。符号可以用作属性名，但无论是在代码还是开发控制台中都无法查看和访问它的值，只会显示为诸如Symbol(Symbol.create)这样的值。

虽然符号实际上并非私有属性（通过Object.getOwnPropertySymbols(..)便可以公开获得对象中的所有符号），但它却主要用于私有或特殊属性。很多开发人员喜欢用它来替代有下划线（）前缀的属性，而下划线前缀通常用于命名私有或特殊属性。

ES6中有一些预定义符号，以Symbol的静态属性形式出现，如Symbol.create、Symbol. iterator等，可以这样来使用：`obj[Symbol.iterator] = function(){ /＊..＊/ };`。

### 原生函数原型

原生构造函数有自己的．prototype对象，如Array.prototype、String.prototype等。有些原生原型（native prototype）并非普通对象，我们甚至可以修改它们。

```
typeof Function.prototype; // "function"
Function.prototype(); // 空函数！
RegExp.prototype.toString();  // "/(? :)/"——空正则表达式
"abc".match( RegExp.prototype ); // [""]
```
Function.prototype是一个空函数，RegExp.prototype是一个“空”的正则表达式（无任何匹配），而Array.prototype是一个空数组。对未赋值的变量来说，它们是很好的默认值。这种方法的一个好处是．prototype已被创建并且仅创建一次。如果变量默认值随后会被更改，那就不要使用Array.prototype。

## 强制类型转换

### 值类型转换

将值从一种类型转换为另一种类型通常称为类型转换（type casting）。在动态类型语言的运行时发生的类型转换叫做强制类型转换（coercion）。JavaScript中的强制类型转换总是返回标量基本类型值，如string, number, 或 boolean。

### ToString

#### ToString操作

抽象操作ToString负责处理非字符串到字符串的强制类型转换。

基本类型值的字符串化规则为：null转换为"null"、undefined转换为"undefined"、true转换为"true"。数字的字符串化则遵循通用规则，极小和极大的数字使用指数形式。

对**普通对象**来说，除非自行定义，否则toString()（Object.prototype.toString()）返回内部属性[[Class]]的值，如"[object Object]"。如果对象有自己的toString()方法，字符串化时就会调用该方法并使用其返回值。String(..)与toString()返回值一致。对**函数**toString()会返回函数代码。

对象强制类型转换为string是通过ToPrimitive抽象操作来完成的。

#### JSON字符串化

对大多数简单值来说，JSON字符串化和toString()的效果基本相同，只不过序列化的结果总是字符串。

undefined、function、symbol（ES6+）和包含循环引用（对象之间相互引用，形成一个无限循环）的对象都不符合JSON结构标准，其他支持JSON的语言无法处理它们。JSON.stringify(..)在对象中遇到undefined、function和symbol时会自动将其忽略，在数组中则会返回null（以保证单元位置不变）。

如果对象中定义了toJSON()方法，JSON字符串化时会首先调用该方法，然后用它的返回值来进行序列化。如果要对含有非法JSON值的对象做字符串化，或者对象中的某些值无法被序列化时，就需要定义toJSON()方法来返回一个安全的JSON值。toJSON()应该“返回一个能够被字符串化的安全的JSON值”，而不是“返回一个JSON字符串”。

```
// 自定义的JSON序列化
a.toJSON = function() {
	// 序列化仅包含b
	return { b: this.b };
};
JSON.stringify( a ); // "{"b":42}"
```

我们可以向JSON.stringify(..)传递一个可选参数replacer，它可以是数组或者函数，用来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除。
- 如果replacer是一个数组，那么它必须是一个字符串数组，其中包含序列化要处理的对象的属性名称，除此之外其他的属性则被忽略-深层遍历。
- 如果replacer是一个函数，它会对对象本身调用一次，然后对对象中的每个属性各调用一次，每次传递两个参数，键和值。如果要忽略某个键就返回undefined，否则返回指定的值。

JSON.stringify(..)并不是强制类型转换。在这里介绍是因为它涉及ToString强制类型转换，具体表现在以下两点。
- 字符串、数字、布尔值和null的JSON.stringify(..)规则与ToString基本相同。
- 如果传递给JSON.stringify(..)的对象中定义了toJSON()方法，那么该方法会在字符串化前调用，以便将对象转换为安全的JSON值。

### ToPrimitive

抽象操作ToPrimitive：
- 首先检查该值是否有valueOf()方法
	- 如果有并且返回基本类型值，就使用该值进行强制类型转换。
	- 否则使用toString()的返回值（如果存在）来进行强制类型转换。
		- 如果toString()也不返回基本类型值，会产生TypeError错误。

### ToNumber

抽象操作ToNumber：
- true转换为1、false转换为0、undefined转换为NaN,、**null转换为0**。
- 对以0开头的十六/八/二进制数转换为十进制处理。
- 对象（包括数组）会首先通过ToPrimitive操作被转换为相应的基本类型值。如果返回的是非数字的基本类型值，则再遵循以上规则将其强制转换为数字。

ES5开始，使用Object.create(null)创建的对象[[Prototype]]属性为null，并且没有valueOf()和toString()方法，因此无法进行强制类型转换。

### ToBoolean

JavaScript中的值可以分为以下两类：
- 可以被强制类型转换为false的值
	- undefined
	- null
	- false
	- +0、-0和NaN
	- ""
- 其他（被强制类型转换为true的值）

### 类型转换实践

#### 显示转换

String(..)遵循前面讲过的ToString规则，将值转换为字符串基本类型。Number(..)遵循前面讲过的ToNumber规则，将值转换为数字基本类型。

if(..).．这样的布尔值上下文中，如果没有使用Boolean(..)和!!，就会自动隐式地进行ToBoolean转换。建议使用Boolean(..)和!!来进行显式转换以便让代码更清晰易读。

#### 隐式转换

隐式强制类型转换的作用是减少冗余，让代码更简洁。

a + ""（隐式）和前面的String(a)（显式）之间有一个细微的差别。根据ToPrimitive抽象操作规则，a + ""会对a调用valueOf()方法，然后通过ToString抽象操作将返回值转换为字符串。而String(a)则是直接调用ToString()。

```
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};
a + "";         // "42"
String( a );    // "4"
```
-是数字减法运算符，因此a -0会将a强制类型转换为数字。也可以使用a ＊ 1和a /1，因为这两个运算符也只适用于数字，只不过这样的用法不太常见。

**发生布尔值隐式强制类型转换条件**
- if (..)语句中的条件判断表达式。
- for ( .. ; .. ; .. )语句中的条件判断表达式（第二个）。
- while (..)和do..while(..)循环中的条件判断表达式。
- ? ：中的条件判断表达式。
- 逻辑运算符||（逻辑或）和&&（逻辑与）左边的操作数（作为条件判断表达式）。


##### +运算符

如果**某个操作数是字符串**或者能够通过**以下步骤转换为字符串**的话，+将进行拼接操作。否则执行数字加法。
- 如果其中一个操作数为简单值，，将其转为字符串。
- 如果其中一个操作数是对象（包括数组），则首先对其调用ToPrimitive抽象操作。

一个坑常常被提到，即[] + {}和{} + []，它们返回不同的结果，分别是"[object Object]"和0。对于{} + []，{}被看作代码块，不过涉及赋值运算，{} + []等于"[object Object]"。`{}+[] === [] + {}`。

#### 一元+操作符
在JavaScript开源社区中，**一元运算+**被普遍认为是显式强制类型转换。一元运算符+的另一个常见用途是将日期（Date）对象强制类型转换为数字`+new Date()`

#### 字符运算符

字位运算符只适用于32位整数，运算符会强制操作数使用32位格式。这是通过抽象操作ToInt32来实现的。ToInt32首先执行ToNumber强制类型转换，比如"123"会先被转换为123，然后再执行ToInt32。

```
0 | -0;         // 0
0 | NaN;        // 0
0 | Infinity;   // 0
0 | -Infinity;  // 0
```
以上这些特殊数字无法以32位格式呈现（因为它们来自64位IEEE 754标准），因此ToInt32返回0。

**~运算符**首先将值强制类型转换为32位数字，然后执行字位操作“非”（对每一个字位进行反转）。`~-1`是唯一返回0的数字取反操作。`~`可以配合`indexOf()`一起使用`if(~a.indexOf('key'))`。

一些开发人员使用**~~运算符**来截除数字值的小数部分，以为这和Math.floor(..)的效果一样，实际上并非如此。
\~\~中的第一个\~执行ToInt32并反转字位，然后第二个~再进行一次字位反转，即将所有字位反转回原值，最后得到的仍然是ToInt32的结果。对\~\~我们要多加注意。首先它只适用于32位数字，更重要的是它对负数的处理与Math. floor(..)不同。

```
Math.floor( -49.6 );    // -50
~~-49.6;                // -49
```

#### parseInt(..)

从ES5开始parseInt(..)默认转换为十进制数，除非另外指定。如果你的代码需要在ES5之前的环境运行，请记得将第二个参数设置为10。

```
var a = "42";
var b = "42px";

Number( a );    // 42
parseInt( a );  // 42
Number( b );    // NaN
parseInt( b );  // 42
```
parseInt允许字符串中含有非数字字符，解析按从左到右的顺序，如果遇到非数字字符就停止。而转换不允许出现非数字字符，否则会失败并返回NaN。

**parseInt(..)针对的是字符串值**。非字符串参数会首先被强制类型**转换为字符串**，依赖这样的隐式强制类型转换并非上策，应该避免向parseInt(..)传递非字符串参数。

```
parseInt( 1/0, 19 ); // 18
```
parseInt(1/0, 19)实际上是parseInt("Infinity", 19)。第一个字符是"I"，以19为基数时值为18。

```
parseInt( 0.000008 );       // 0   ("0" 来自于 "0.000008")
parseInt( 0.0000008 );      // 8   ("8" 来自于 "8e-7")
parseInt( false, 16 );      // 250 ("fa" 来自于 "false")
parseInt( parseInt, 16 );   // 15  ("f" 来自于 "function..")
parseInt( "0x10" );         // 16
parseInt( "103", 2 );       // 2
```

#### Symbol的类型转换

ES6中引入了符号类型，它的强制类型转换有一个坑，在这里有必要提一下。ES6允许从符号到字符串的显式强制类型转换，然而隐式强制类型转换会产生错误。

符号不能够被强制类型转换为数字（显式和隐式都会产生错误），但可以被强制类型转换为布尔值（显式和隐式结果都是true）。

由于规则缺乏一致性，我们要对ES6中符号的强制类型转换多加小心。

#### ==与===

==和===都会检查操作数的类型。区别在于操作数类型不同时它们的处理方式不同。==允许在相等比较中进行强制类型转换，而===不允许。

比较两个对象的时候，==和===的工作原理是一样的。

==在比较两个不同类型的值时会发生隐式强制类型转换，会将其中之一或两者都转换为相同的类型后再进行比较。

##### 宽松相等

- **字符串和数字相等比较**
	- 如果Type(x)是数字，Type(y)是字符串，则返回x == ToNumber(y)的结果。
	- 如果Type(x)是字符串，Type(y)是数字，则返回ToNumber(x) == y的结果。
- **其他类型与布尔类型相等比较**
	- 如果Type(x)是布尔类型，则返回ToNumber(x) == y的结果；
	- 如果Type(y)是布尔类型，则返回x == ToNumber(y)的结果。
- **null与undefined相等比较**
	- null与undefined相等
	- null与undefined与其它假值不相等
- **对象与非对象相等比较**
	- 如果Type(x)是字符串或数字，Type(y)是对象，则返回x == ToPrimitive(y)的结果；
	- 如果Type(x)是对象，Type(y)是字符串或数字，则返回ToPrimitive(x) == y的结果。

Object(null)和Object()均返回一个常规对象。NaN能够被封装为数字封装对象，但拆封之后NaN == NaN返回false，因为NaN不等于NaN。

##### 非常见情况

```
// 示例一
var i = 2;
Number.prototype.valueOf = function() {
	return i++;
};
var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "Yep, this happened." );
}

// 示例二
[] == ![]	// true
"" == [null];	// true
[null, undefined, null] == ",,"	// true
0 == '\n   \t '	// true
```

以下两个原则可以让我们有效地避免出错：
- 如果两边的值中有true或者false，千万不要使用==。
- 如果两边的值中有[]、""或者0，尽量不要使用==。

#### 抽象关系比较

**“抽象关系比较”，分为两个部分**
- 比较双方都是字符串则按字母顺序来进行比较。
- 其他情况。比较双方首先调用ToPrimitive，如果结果出现非字符串，就根据ToNumber规则将双方强制类型转换为数字来进行比较。

```
// 示例一
var a = { b: 42 };
var b = { b: 43 };
a < b;  // false
a == b; // false
a > b;  // false

a <= b; // true
a >= b; // true

// 示例二
var a = [ 42 ];
var b = "043";
a < b;                      // false

// 示例三
42 < '043'	// true
```

## 语法

### 语句

语句都有一个结果值（statement completion value, undefined也算）。以赋值表达式b = a为例，其结果值是赋给b的值，但规范定义var的结果值是undefined。如果在控制台中输入var a = 42会得到结果值undefined，而非42。

{ }在不同情况下的意思不尽相同，可以是语句块、对象常量、解构赋值（ES6）或者命名函数参数（ES6）。

代码块{ .. }的结果值是其最后一个语句/表达式的结果。语法不允许我们获得语句的结果值并将其赋值给另一个变量（至少目前不行）。不过，可以使用万恶的eval(..)（又读作“evil”）来获得结果值。

ES7规范有一项“do表达式”（do expression）提案：

```
var a, b;
a = do {
	if (true) {
		b = 4 + 38;
  }
};
a;  // 42
```

++无法直接在42这样的值上产生副作用，++42会报错。b = ( a++, a )可以将a+1然后赋值给b。

链式赋值常常被误用，例如var a = b = 42，看似和a = b = 42，实则不然。如果变量b没有在作用域中象var b这样声明过，则var a = b =42不会对变量b进行声明。在严格模式中这样会产生错误，或者会无意中创建一个全局变量。

事实上JavaScript没有else if，但if和else只包含单条语句的时候可以省略代码块的{ }。

### 标签语句

带标签的循环/代码块十分少见，也不建议使用。

```
// 标签为foo的循环
foo: for (var i=0; i<4; i++) {
    for (var j=0; j<4; j++) {
      if ((i ＊ j) >= 3) {
          console.log( "stopping! ", i, j );
          break foo;
      }
      console.log( i, j );
    }
}￼
```
break foo不是指“跳转到标签foo所在位置继续执行”，而是“跳出标签foo所在的循环/代码块，继续执行后面的代码”。因此它并非传统意义上的goto。


JSON-P（将JSON数据封装为函数调用，比如foo({"a":42})）通过将JSON数据传递给函数来实现对其的访问。它能将JSON转换为合法的JavaScript语法。

### 对象解构

ES6开始，{ .. }也可用于“解构赋值”。

```
function getData() {
	// ..
	return {
	  a: 42,
	  b: "foo"
 	};
}
var { a, b } = getData();

var { a: c, b: d } = { a: 1, b : 2 };
console.log(c, d) // 1 2
```
{ a, b }实际上是{ a: a, b: b }的简化版本，两者均可，只不过{ a, b }更简洁。

es6中支持import引入模块，也支持类似上面写法，但是语法稍有不同：`import { a as c, d as b } from file`。

### else if和可选代码块

事实上JavaScript没有else if，但if和else只包含单条语句的时候可以省略代码块的{ }。所以我们经常用到的else if实际上是这样的：

```
if (a) {
	// ..
} else {
	if (b) {
  	// ..
	}
	else {
	  	// ..
	}
}
```

### 运算符优先级

用，来连接一系列语句的时候，它的优先级最低，其他操作数的优先级都比它高。

&&运算符的优先级高于||，而||的优先级又高于？ :。

如果&&是右关联的话会被处理为a && (b && c)。但这并不意味着c会在b之前执行。右关联不是指从右往左执行，而是指从右往左组合。情况并非总是这样。一些运算符在左关联和右关联时的表现截然不同。比如？：（即三元运算符或者条件运算符）。

**?:是右关联，它的组合顺序是以下哪一种呢？**
- a ? b : (c ? d : e)
- (a ? b : c) ? d : e
答案是a ? b : (c ? d : e)。

如果运算符优先级/关联规则能够令代码更为简洁，就使用运算符优先级/关联规则；而如果( )有助于提高代码可读性，就使用( )。

### 自动分号

有时JavaScript会自动为代码行补上缺失的分号，即自动分号插入（Automatic Semicolon Insertion, ASI）。

ASI只在换行符处起作用，而不会在代码行的中间插入分号。如果JavaScript解析器发现代码行可能因为缺失分号而导致错误，那么它就会自动补上分号。并且，只有在代码行末尾与换行符之间除了空格和注释之外没有别的内容时，它才会这样做。

```
var a = 42, b
c;
```
如果b和c之间出现，的话（即使另起一行）, c会被作为var语句的一部分来处理。

```
var a = 42;
do {
	// ..
} while (a) // <-- 这里应该有;
a;
```
语法规定do..while循环后面必须带；，而while和for循环后则不需要。大多数开发人员都不记得这一点，此时ASI就会自动补上分号。语句代码块结尾不用带；，所以不需要用到ASI。其他涉及ASI的情况是break、continue、return和yield（ES6）等关键字。

ASI实际上是一个**“纠错”机制**。这里的错误是指解析器错误。换句话说，ASI的目的在于提高解析器的容错性。因此，建议在所有需要的地方加上分号，将对ASI的依赖降到最低。

### 错误

#### 错误类型
JavaScript中有很多错误类型，分为两大类：早期错误（编译时错误，无法被捕获）和运行时错误（可以通过try..catch来捕获）。所有语法错误都是早期错误，程序有语法错误则无法运行。

#### TDZ(暂时性死区)

ES6规范定义了一个新概念，叫作TDZ（Temporal Dead Zone，暂时性死区）。TDZ指的是由于代码中的变量还没有初始化而不能被引用的情况。对此，最直观的例子是ES6规范中的let块作用域。

```
{
	typeof b;   // undefined
	typeof a;   // ReferenceError! (TDZ)
	a = 2;      // ReferenceError!
	let a;
}
```
对未声明变量使用typeof不会产生错误，但在TDZ中却会报错。

```
var b = 3;
function foo( a = 42, b = a + b + 5 ) {
	// ..
}
```
b = a + b + 5在参数b（=右边的b，而不是函数外的那个）的TDZ中访问b，所以会出错。

在ES6中，如果参数被省略或者值为undefined，则取该参数的默认值。ES6参数默认值会导致arguments数组和相对应的命名参数之间出现偏差，ES5也会出现这种情况。arguments数组已经被废止。

### try...finally

finally中的代码总是会在try之后执行，如果有catch的话则在catch之后执行。也可以将finally中的代码看作一个回调函数，即无论出现什么情况最后一定会被调用。

如果try中有return语句会出现什么情况呢?

```
function foo() {
	try {
    return 42;
  }
	finally {
    console.log( "Hello" );
  }
  console.log( "never runs" );
}
console.log( foo() );
// Hello
// 42
```
这里return 42先执行，并将foo()函数的返回值设置为42。然后try执行完毕，接着执行finally。最后foo()函数执行完毕，console.log(..)显示返回值。try中的throw也是如此。

如果finally中抛出异常（无论是有意还是无意），函数就会在此处终止。如果此前try中已经有return设置了返回值，则该值会被丢弃。通常来说，在函数中省略return的结果和return；及return undefined；是一样的，但是在finally中省略return则会返回前面的return设定的返回值。

### switch

case表达式的匹配算法与===相同。有时可能会需要通过强制类型转换来进行相等比较（即==，参见第4章），这时就需要做一些特殊处理：

```
var a = "42";
switch (true) {
	case a == 10:
    console.log( "10 or '10'" );
    break;
  case a == 42;
    console.log( "42 or '42'" );
    break;
  default:
    // 永远执行不到这里
}
// 42 or '42'
```

```
var a = 10;
switch (a) {
	case 1:
	case 2:
    // 永远执行不到这里
  default:
		console.log( "default" );
	case 3:
		console.log( "3" );
		break;
	case 4:
		console.log( "4" );
}
// default
// 3
```
上例中的代码是这样执行的，首先遍历并找到所有匹配的case，如果没有匹配则执行default中的代码。因为其中没有break，所以继续执行已经遍历过的case 3代码块，直到break为止。

## 混合环境Javascript

JavaScript语言本身有一个统一的标准，在所有浏览器/引擎中的实现也是可靠的。但是JavaScript很少独立运行。通常运行环境中还有第三方代码，有时代码甚至会运行在浏览器之外的引擎/环境中。

### Annex B（ECMAScript）

JavaScript语言的官方名称是ECMAScript（指的是管理它的ECMA标准）。官方ECMAScript规范包括Annex B，其中介绍了由于浏览器兼容性问题导致Javascript实现的与官方规范的差异。

如果代码只在浏览器中运行，则无需考虑这些差异。否则（如果代码也在Node.js、Rhino等环境中运行），或者你也不确定的时候，就需要小心对待。

下面是主要的兼容性差异：
- 在非严格模式中允许八进制数值常量存在，如0123（即十进制的83）。
- window.escape(..)和window.unescape(..)让你能够转义（escape）和回转（unescape）带有%分隔符的十六进制字符串。例如，window.escape( "? foo=97%&bar=3%" )的结果为"%3Ffoo%3D97%25%26bar%3D3%25"。
- String.prototype.substr和String.prototype.substring十分相似，除了后者的第二个参数是结束位置索引（非自包含），而substr的第二个参数是长度（需要包含的字符数）。

### 宿主对象

“宿主对象”（包括内建对象和函数）是由宿主环境（浏览器等）创建并提供给JavaScript引擎的变量。

针对运行环境进行编码时，宿主对象扮演着一个十分关键的角色，但要特别注意其行为特性，因为它们常常有别于普通的JavaScript object。比如，一些对象在强制转换为boolean时，会意外地成为假值而非真值。其他需要注意的宿主对象的行为差异有：
- 无法访问正常的object内建方法，如toString();
- 无法写覆盖；
- 包含一些预定义的只读属性；
- 包含无法将this重载为其他对象的方法；
- 其他……

在我们经常打交道的宿主对象中，console及其各种方法（log(..)、error(..)等）是比较值得一提的。console对象由宿主环境提供，以便从代码中输出各种值。

### 全局DOM对象

由于浏览器演进的历史遗留问题，在创建带有id属性的DOM元素时也会创建同名的全局变量。

如上HTML页面中的内容也会产生全局变量，这也是尽量不要使用全局变量的一个原因。如果确实要用，也要确保变量名的唯一性，从而避免与其他地方的变量产生冲突，包括HTML和其他第三方代码。

### 原生原型

**不要扩展原生原型。**如果向Array.prototype中加入新的方法和属性，假设它们确实有用，设计和命名都很得当，那它最后很有可能会被加入到JavaScript规范当中。这样一来你所做的扩展就会与之冲突。

### script标签

`<script src=..></script>`来加载的文件，或者使用`<script> .. </script>`来包含的内联代码（inline-code）是相互独立的JavaScript程序还是一个整体呢？

答案（也许会令人惊讶）是它们的运行方式更像是相互独立的JavaScript程序，但是并非总是如此。它们共享global对象（在浏览器中则是window），也就是说这些文件中的代码在共享的命名空间中运行，并相互交互。但是全局变量作用域的提升机制在这些边界中不适用。

如果script中的代码（无论是内联代码还是外部代码）发生错误，它会像独立的JavaScript程序那样停止，但是后续的script中的代码（仍然共享global）依然会接着运行，不会受影响。

内联代码和外部文件中的代码之间有一个区别，即在内联代码中不可以出现`</script>`字符串，一旦出现即被视为代码块结束。常用的变通方法是：
`"</sc" + "ript>";`

另外需要注意的一点是，我们是根据代码文件的字符集属性（UTF-8、ISO-8859-8等）来解析外部文件中的代码（或者默认字符集），而内联代码则使用其所在页面文件的字符集（或者默认字符集）。

### 实现中的限制

JavaScript规范对于函数中参数的个数，以及字符串常量的长度等并没有限制；但是由于JavaScript引擎实现各异，规范在某些地方有一些限制。

已知的限制：
- 字符串常量中允许的最大字符数（并非只是针对字符串值）；
- 可以作为参数传递到函数中的数据大小（也称为栈大小，以字节为单位）；
- 函数声明中的参数个数；
- 未经优化的调用栈（例如递归）的最大层数，即函数调用链的最大长度；
- JavaScript程序以阻塞方式在浏览器中运行的最长时间（秒）；
- 变量名的最大长度。

## 资料

* [You Don't Know JS: Types & Grammar](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20&%20grammar/README.md#you-dont-know-js-types--grammar)
* [Why `null >= 0 && null <= 0` but not `null == 0`?](https://stackoverflow.com/questions/2910495/why-null-0-null-0-but-not-null-0)