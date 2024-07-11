\#ES6 <https://www.bookstack.cn/read/es6-3rd/spilt.4.docs-generator.md>

## let var const以及作用域

### let var const

    - 状态提升：在变量声明前，使用变量进行操作；
    - 暂时性死区：在块级作用域内，只要存在let、const声明了该变量，那么在该作用域内该变量只能在声明的位置后面使用。
    - let 声明的变量只在代码块内有效；只能够在声明后使用，声明前使用会报referenceError；不允许在同一作用域重复声明同一变量；只能够声明在当前作用域的顶层；如果区块中存在let和const命令，区块内命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错，哪怕在区块外的作用域中声明有相同名称的变量；
    - var 声明的变量在全局有效；能够在声明前使用，值为undefined；允许在同一作用域重复声明同一变量；
    - const 声明一个只读的常量（需要注意如果是对象则存储的地址，内部的值是可以改变的）；const在声明常量时，就必须初始化；const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用；同样存在暂时性死区；const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了；如果想要将一个对象冻结不能够进行操作可以使用const foo = Object.freeze({});
    -ES6声明变量的六种方法：var、let、const、function、class、import

### 块级作用域

    - ES5只有全局作用域和函数作用域，没有块级作用域。
    - ES5不允许在块作用域中声明方法，但考虑由于浏览器没有这方面限制，在编解码过程中块内声明的function会被提到块外；
    - 考虑到ES5和6作用域之间的差异较大，建议function写成函数表达式而不是函数声明语句: let a = function a(){return a;}
    - ES6块级作用域必须要有大括号；
    - 函数只能够声明在作用域的顶层；

### 顶层对象的属性

    - 顶层对象在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的;ES6 一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。
    ```
    var a = 1;
    // 如果在 Node 的 REPL 环境，可以写成 global.a
    // 或者采用通用方法，写成 this.a
    window.a // 1
    let b = 1;
    window.b // undefined
    ```
    - JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的。

    浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
    浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
    Node 里面，顶层对象是global，但其他环境都不支持。
    同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用this变量，但是有局限性。

    全局环境中，this会返回顶层对象。但是，Node 模块和 ES6 模块中，this返回的是当前模块。
    函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
    不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全策略），那么eval、new Function这些方法都可能无法使用。
    - global-this:ES2020 在语言标准的层面，引入globalThis作为顶层对象。也就是说，任何环境下，globalThis都是存在的，都可以从它拿到顶层对象，指向全局环境下的this。

## 变量的解构赋值

    - 按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
    - 数组进行解构赋值按照次序排列进行；
    - 对象进行解构赋值按照变量名称进行赋值；
    - 函数参数也能够进行解构赋值;
    - 字符串也可以进行解构赋值，字符串会被转换成为类似数组的对象，可以作为数组进行结构；
    - 数组的length属性也可以通过let {length : len} = 'hello';进行解构；
    - 解构赋值若等号右侧不是对象或数组，则先转换为对象进行解析；
    - 解构赋值允许指定默认值，但对应的数组值必须严格等于（===）undefined；
    - 解构赋值失败值为undefined；
    - 解构赋值语句中不能够使用圆括号的情况();1、变量声明语句不能够使用圆括号；2、函数参数也属于变量声明语句，不能够使用圆括号；3、赋值语句中模式部分（冒号左侧）不能够使用圆括号；
    ```
    //数组解构
    let [foo, [[bar], baz]] = [1, [[2], 3]];
    // Generator 函数，原生具有 Iterator 接口。 解构赋值会依次从这个接口获取值。
    function* fibs() {
    let a = 0;
    let b = 1;
    while (true) {
    	yield a;
    	[a, b] = [b, a + b];
    	}
    }
    let [first, second, third, fourth, fifth, sixth] = fibs();

    //对象解构
    let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
    let { log, sin, cos } = Math;
    const { log } = console;
    log('hello') // hello

    //如果变量名与属性名不同
    let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
    //结果：baz // "aaa"
    //对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。
    //foo是匹配的模式，baz才是变量。真正被赋值的是变量baz，而不是模式foo。
    //理解方式：let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };

    //嵌套赋值
    let obj = {
    	p: [
    		'Hello',
    		{ y: 'World' }
    	]
    };
    let { p: [x, { y }] } = obj;
    x // "Hello"
    y // "World"
    //注意，这时p是模式，不是变量，因此不会被赋值。如果p也要作为变量赋值，可以写成下面这样。
    let { p, p: [x, { y }] } = obj;


    const node = {
    	loc: {
    		start: {
    		line: 1,
    		column: 5
    		}
    	}
    };
    let { loc, loc: { start }, loc: { start: { line }} } = node;
    line // 1
    loc  // Object {start: Object}
    start // Object {line: 1, column: 5}
    //冒号左侧为模式用于匹配


    //如果要将一个已经声明的变量用于解构赋值，必须非常小心。
    let x;
    {x} = {x: 1};
    //{x}理解成一个代码块，从而发生语法错误。需要确保大括号不写在行首如：
    ({x} = {x: 1});


    //由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。
    let arr = [1, 2, 3];
    let {0 : first, [arr.length - 1] : last} = arr;
    first // 1
    last // 3
    ```

## ES6字符串方法

    - String.fromCodePoint()：增强ES5 String.fromCharCode()不能识别大于0xFFFF的字符；
    - String.raw()：该方法返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。无脑在斜杠前加斜杠；
    - codePointAt()：返回的是码点的十进制值，如果想要十六进制的值，可以使用toString()方法转换一下。
    - normalize()：用于处理两个字符编码的合成结果；用于判断等；
    - includes()：返回布尔值，表示是否找到了参数字符串。第二个参数，表示开始搜索的位置。
    - startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。第二个参数，表示开始搜索的位置。
    - endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。第二个参数，表示开始搜索的位置。
    - repeat()：重复字符串n次，如果是n是小数则取整，负数或Infinity则报错、NaN等同于0、字符串则转换为数字；
    - padStart()、padEnd()：用于补全字符串长度；若补全字符+元字符长于设定长度，则截取补全字符以满足条件；'x'.padStart(5, 'ab') // 'ababx'
    - trimStart()，trimEnd()，trim();清除空格
    - matchAll()：方法返回一个正则表达式在当前字符串的所有匹配。

## 正则表达式扩展

    - 字符串对象共有 4 个方法，可以使用正则表达式：match()、replace()、search()和split()。
    ES6 将这 4 个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。
    - \p{...}正向匹配和\P{...} 反向匹配：匹配的是字符的Unicade值；这两种类只对 Unicode 有效，所以使用的时候一定要加上u修饰符。
    ```
    // 匹配所有数字
    const regex = /^\p{Number}+$/u;
    regex.test('²³¹¼½¾') // true
    regex.test('㉛㉜㉝') // true
    regex.test('ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫ') // true
    const regex = /^\p{Decimal_Number}+$/u;
    regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true
    // 匹配所有空格
    \p{White_Space}
    // 匹配各种文字的所有字母，等同于 Unicode 版的 \w
    [\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]
    // 匹配各种文字的所有非字母的字符，等同于 Unicode 版的 \W
    [^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]
    // 匹配 Emoji
    /\p{Emoji_Modifier_Base}\p{Emoji_Modifier}?|\p{Emoji_Presentation}|\p{Emoji}\uFE0F/gu
    // 匹配所有的箭头字符
    const regexArrows = /^\p{Block=Arrows}+$/u;
    regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
    ```
    - 具名组匹配（正则表达式）
    ```
    //1、圆括号进行分组匹配，使用exec进行结果提取
    const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
    const matchObj = RE_DATE.exec('1999-12-31');
    const year = matchObj[1]; // 1999
    const month = matchObj[2]; // 12
    const day = matchObj[3]; // 31

    //2、具名组匹配?<groupName>，为每个正则匹配组命名，可以根据命名提取结果而不是数组下标，若具名组未匹配到值对应键名依旧存在，值为undefined
    const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
    const matchObj = RE_DATE.exec('1999-12-31');
    const year = matchObj.groups.year; // 1999
    const month = matchObj.groups.month; // 12
    const day = matchObj.groups.day; // 31

    //3、具名匹配可以通过解构赋值直接获取结果
    let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
    one  // foo
    two  // bar

    //4、可以通过$<组名>进行字符串替换
    let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;
    '2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
    // '02/01/2015'


    //5、可以通过\k<组名>的写法在正则表达式内部引用具名组匹配, \1 数组引用依然有效
    const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
    RE_TWICE.test('abc!abc') // true
    RE_TWICE.test('abc!ab') // false

    const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
    RE_TWICE.test('abc!abc!abc') // true
    RE_TWICE.test('abc!abc!ab') // false
    ```

    - 正则匹配索引
    ```
    //1、正则实例的exec()方法返回的结果的index属性，可以获取整个匹配结果的开始位置，indices属性可以获取匹配结果的开始和结束为止[1,3)位置从0开始左闭右开。若有多个匹配结果，indices的数组会包含多个数组成员。
    const text = 'zabbcdef';
    const re = /ab+(cd(ef))/;
    const result = re.exec(text);
    result.indices // [ [1, 8], [4, 8], [6, 8] ]

    //2、如果使用具名表达式进行匹配，indices属性数组会有一个groups属性，该属性是一个对象，可以从该对象使用具名组名称获取具名组匹配的开始位置和结束位置。若未匹配成功，则为undefined。
    const text = 'zabbcdef';
    const re = /ab+(?<Z>cd)/;
    const result = re.exec(text);
    result.indices.groups // { Z: [ 4, 6 ] }
    ```

    - String.prototype.matchAll() 正则匹配字符串中所有符合条件的字符
    ```
    //1、正则表达式在字符串里面有多个匹配，可以使用g修饰符或者y修饰符，使用循环逐一取出。
    var regex = /t(e)(st(\d?))/g;
    var string = 'test1test2test3';
    var matches = [];
    var match;
    while (match = regex.exec(string)) {
    matches.push(match);
    }
    matches
    // [
    //   ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"],
    //   ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"],
    //   ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
    // ]

    //2、string.matchAll(regex)，可以一次性取出所有匹配，返回一个遍历器(Iterator)。遍历器可以通过for...of循环取出。相较于返回数组，遍历器较为节省资源。
    const string = 'test1test2test3';
    // g 修饰符加不加都可以
    const regex = /t(e)(st(\d?))/g;
    for (const match of string.matchAll(regex)) {

console.log(match);
}
// \["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// \["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// \["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]

    //遍历器转换为数组
    // 转为数组方法一
    [...string.matchAll(regex)]
    // 转为数组方法二
    Array.from(string.matchAll(regex))
    ```
    - U 修饰符
    ES6对正则表达式添加了u修饰符，含义为“Unicode模式”，用来正确处理大于\uFFF的Unicode字符。也就是四字节的UTF-16编码。
    ```
    //1、点字符，（.）在正则表达式中代表除了换行符以外的任意单个字符。对于码点大于0xFFFF的Unicode字符不能够识别，必须加上u修饰符。
    var s = '𠮷';
    /^.$/.test(s) // false
    /^.$/u.test(s) // true

    //2、Unicode字符表示法，ES6 新增了使用大括号表示 Unicode 字符，这种表示法在正则表达式中必须加上u修饰符，才能识别当中的大括号，否则会被解读为量词。
    /\u{61}/.test('a') // false
    /\u{61}/u.test('a') // true
    /\u{20BB7}/u.test('𠮷') // true

    //3、量词，使用u修饰符后，所有量词都会正确识别码点大于0XFFFF的Unicode字符。
    /a{2}/.test('aa') // true
    /a{2}/u.test('aa') // true
    /𠮷{2}/.test('𠮷𠮷') // false
    /𠮷{2}/u.test('𠮷𠮷') // true

    //4、预定义模式，只有加了u修饰符，预定义模式才能够识别码点大于0xFFFF的Unicode字符
    /^\S$/.test('𠮷') // false
    /^\S$/u.test('𠮷') // true

    //5、i修饰符，有些 Unicode 字符的编码不同，但是字型很相近，比如，\u004B与\u212A都是大写的K。
    /[a-z]/i.test('\u212A') // false
    /[a-z]/iu.test('\u212A') // true

    //6、转义，没有u修饰符的情况下，正则中没有定义的转义（如逗号的转义\,）无效，而在u模式会报错。
    /\,/ // /\,/
    /\,/u // 报错
    ```

    - RegExp.prototype.unicode属性，正则实例对象新增unicode属性，表示是否设置了u修饰符。
    ```
    const r1 = /hello/;
    const r2 = /hello/u;
    r1.unicode // false
    r2.unicode // true
    ```

    - y修饰符，除了u修饰符，ES6 还为正则表达式添加了y修饰符，叫做“粘连”（sticky）修饰符。y同样是全局匹配，y修饰符后一次匹配会从上一次匹配成功的下一个位置开始，也就是y匹配字符成功后的剩余字符的起始处开始匹配，匹配方式为从起始位置开始匹配成功则返回，失败则为null，和g修饰符不同，g若起始处不匹配，后续有匹配则可。y起始处必须匹配。实际上，y修饰符号隐含了头部匹配的标志^。y修饰符确保字符连贯匹配，确保匹配结果不会丢失元字符。
    ```
    var s = 'aaa_aa_a';
    var r1 = /a+/g;
    var r2 = /a+/y;
    r1.exec(s) // ["aaa"]
    r2.exec(s) // ["aaa"]
    r1.exec(s) // ["aa"]
    r2.exec(s) // null

    //单单一个y修饰符对match方法，只能返回第一个匹配，必须与g修饰符联用，才能返回所有匹配。
    'a1a2a3'.match(/a\d/y) // ["a1"]
    'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
    ```

    - RegExp.protorype.sticky 可以查看正则实例是否设置了y修饰符。
    ```
    var r = /hello\d/y;
    r.sticky // true
    ```

    - RegExp.prototype.flags 可以获得正则表达式的修饰符
    ```
    // ES5 的 source 属性
    // 返回正则表达式的正文
    /abc/ig.source
    // "abc"
    // ES6 的 flags 属性
    // 返回正则表达式的修饰符
    /abc/ig.flags
    // 'gi'
    ```

    - dotAll模式，即点(dot)代表一切字符。一般情况下，点(.)代表任意字符，但有两个例外，一是：四个字节的UTF-16字符，可以使用u解决；另一个是行终止符;
    ```
    //使用s修饰符，可以使.匹配任意单个字符
    const re = /foo.bar/s;
    // 另一种写法
    // const re = new RegExp('foo.bar', 's');
    re.test('foo\nbar') // true
    re.dotAll // true
    re.flags // 's'
    ```

## 数值的扩展

    - Number.isFinite()，用于判断参数类型是否是有限的。 Number.isNaN()，用于判断参数类型是不是NaN。ES6以后不对非Number的参数进行类型转换，若参数不为Number类型为false。
    ```
    Number.isFinite(15); // true
    Number.isFinite(0.8); // true
    Number.isFinite(NaN); // false
    Number.isFinite(Infinity); // false
    Number.isFinite(-Infinity); // false
    Number.isFinite('foo'); // false
    Number.isFinite('15'); // false
    Number.isFinite(true); // false

    Number.isNaN(NaN) // true
    Number.isNaN(15) // false
    Number.isNaN('15') // false
    Number.isNaN(true) // false
    Number.isNaN(9/NaN) // true
    Number.isNaN('true' / 0) // true
    Number.isNaN('true' / 'true') // true
    ```

    - Number.parseInt(), Number.parseFloat(),ES6 将全局方法parseInt()和parseFloat()，移植到Number对象上面，行为完全保持不变。
    ```
    // ES5的写法
    parseInt('12.34') // 12
    parseFloat('123.45#') // 123.45
    // ES6的写法
    Number.parseInt('12.34') // 12
    Number.parseFloat('123.45#') // 123.45
    ```

    - Number.isInteger(),用于判断一个数值是否是整数；
    ```
    //JavaScript 内部，整数和浮点数采用的是同样的储存方法，所以 25 和 25.0 被视为同一个值。

    Number.isInteger(25) // true
    Number.isInteger(25.0) // true

    //如果参数不是数值，Number.isInteger返回false。

    Number.isInteger() // false
    Number.isInteger(null) // false
    Number.isInteger('15') // false
    Number.isInteger(true) // false

    //注意，由于 JavaScript 采用 IEEE 754 标准，数值存储为64位双精度格式，数值精度最多可以达到 53 个二进制位（1 个隐藏位与 52 个有效位）。如果数值的精度超过这个限度，第54位及后面的位就会被丢弃，这种情况下，Number.isInteger可能会误判。原因就是这个小数的精度达到了小数点后16个十进制位，转成二进制位超过了53个二进制位，导致最后的那个2被丢弃了。

    Number.isInteger(3.0000000000000002) // true

    //类似的情况还有，如果一个数值的绝对值小于Number.MIN_VALUE（5E-324），即小于 JavaScript 能够分辨的最小值，会被自动转为 0。这时，Number.isInteger也会误判。
    Number.isInteger(5E-324) // false
    Number.isInteger(5E-325) // true
    ```

    - Number.EPSILON 它表示 1 与大于 1 的最小浮点数之间的差。用于做误差检测。
    ```
    function withinErrorMargin (left, right) {
    	return Math.abs(left - right) < Number.EPSILON * Math.pow(2, 2);
    }
    0.1 + 0.2 === 0.3 // false
    withinErrorMargin(0.1 + 0.2, 0.3) // true
    1.1 + 1.3 === 2.4 // false
    withinErrorMargin(1.1 + 1.3, 2.4) // true
    ```

    - 安全整数和 Number.isSafeInteger()，JavaScript 能够准确表示的整数范围在-2^53到2^53之间（不含两个端点），超过这个范围，无法精确表示这个值。需要注意的是，不应只验证运算结果是否在安全范围，也应该验证参与计算数值的可靠性。在计算过程中可能会因为超出精度范围，导致在计算机内部已经进行过数值处理。
    ```
    Number.isSafeInteger(9007199254740993)
    // false
    Number.isSafeInteger(990)
    // true
    Number.isSafeInteger(9007199254740993 - 990)
    // true
    9007199254740993 - 990
    // 返回结果 9007199254740002
    // 正确答案应该是 9007199254740003
    ```

    - BigInt() 所有数字都保存成64位浮点数，只能够有整数，数值尾部需要添加n后缀；若使用函数进行数值处理，函数参数值为Number，则需要进行转换；

## 箭头函数

    - 由于箭头函数不存在私有的作用域、上下文，所以箭头函数内的this指向外部作用域。并且由于没有作用域，所以不能够new();
    - function()声明的函数，拥有私有的context，所以this是指向内部对象。

## 数组的扩展

    - 数组扩展运算符 ...[]，外部只能够使用方法的圆括号包裹。使用...以后可以将数组的值解构出来。
    - 数组的空位，ES5大部分情况依旧会认为是空位，ES6则明确将空位转为undefined
    for...of并不会忽略空位。map方法遍历，空位是会跳过的。entries()、keys()、values()、find()和findIndex()会将空位处理成undefined。
    - 扩展运算符背后调用的是遍历器接口（Symbol.iterator），如果一个对象没有部署这个接口，就无法转换。
    - Array.from必须有length属性。因此，任何有length属性的对象，都可以通过Array.from方法转为数组，而此时扩展运算符就无法转换。

## Promise

    - Promise异步编程操作，共有三种状态（pending(进行中)、fulfilled(已成功)、rejected(已失败))
    - Promise 的状态，从进行中变为成功或失败状态之后，状态就固定了，不会再发生改变。
    - Promise实例化的时候，传入的参数是一个函数，函数接收两个参数，参数本身也都为函数(参数由 JavaScript 引擎提供，不用自己部署。)。
    ```javaScript
    // resolve() 将Promise的状态从进行中变为成功状态
    // reject() 将Promise的状态从进行中变为拒绝状态也就是失败状态
    	new Promise((resolve,reject)=>{
    	....code
    	}).then({callback code})
    ```

### Promise.then()

    - 在执行resolve()时，Promise状态会变为fulfilled，会执行 .then 方法。
    - then()方法可以接收两个函数作为参数，第一个参数是resolved状态的回调函数，第二个参数(可选)是rejected状态的回调函数。then方法返回的是一个新的Promise实例。(不建议使用then的第二个参数进行错误处理推荐catch())
    ```javaScript
    	new Promise((resolve,reject)=>{
    		...code
    	})
    		.then(res=>{
    			...code
    			})
    ```

### Promise.catch

    - catch()方法是.then(null, rejection)或.then(undefined, rejection)的别名，用于指定发生错误时的回调函数。
    - 执行 reject 时，Promise 状态从 pending 变为 rejected，会执行 catch 方法，catch 方法接收的也是一个函数，函数中携带一个参数，该参数为 reject(err) 返回的数据。
    - 跟传统的try/catch代码块不同的是，如果没有使用catch()方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，即不会有任何反应。
    - catch()方法返回的还是一个 Promise 对象，因此后面还可以接着调用then()方法。若Promise报错在执行完catch()方法指定的回调函数后，会接着执行then()方法制定的回调函数。若Promise为报错则不执行catch()方法。若catch()方法后面的then()方法出现错误，与前面的catch()方法无关。
    ```javaScript
    	new Promise((resolve,reject)=>{
    		resolve(2+x)
    	})
    	.catch(err=>console.log('error',err))
    	.then(res=>console.log('carry on',res))
    ```
    - 调用resolve()或reject()并不会终结Promise的参数函数的执行。并且resolve()会晚于本轮循环的同步任务，因为resolve是在本轮事件循环的末尾执行。
    ```JavaScript
    	new Promise((resolve,reject)=>{
    		resolve(1)
    		console.log(2)
    		})
    			.then(res=>{
    				console.log(res)
    				});
    	// 2
    	// 1
    ```
    - catch()方法中还能在抛出错误，若后面没有其他的catch()方法，则该错误并不会被捕捉，也不会传递到外层。

### Promise.all()

    - Promise.all()方法接受一个数组作为参数，包装成一个新的 Promise 实例。只有全部数组成员的状态都变成fulfilled，Promise的状态才会变成fulfilled，此时数组成员的返回值组成一个数组，传递给Promise的回调函数。
    - 只要数组成员之中有一个被rejected，Promise的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给Promise的回调函数。

### Promise.race()

    - 接受一个数组作为参数，包装成一个新的 Promise 实例。只要有一个实例率先改变状态，Promise的状态就随之更改，并将率先更改状态的Promise实例返回给回调函数。

### Promise.allSettled()

    - 接受一个数组作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。
    - Promise.allSettled()的返回值状态只能够是fulfilled。返回数组的每个成员都是一个对象，每个对象都有status属性，该属性的值只可能是字符串fulfilled或字符串rejected。fulfilled时，对象有value属性，rejected时有reason属性，对应两种状态的返回值。
    ```javaScript
    	const resolved = Promise.resolve(42);
    	const rejected = Promise.reject(-1);
    	const allSettledPromise = Promise.allSettled([resolved, rejected]);
    	allSettledPromise.then(function (results) {
    	  console.log(results);
    	});
    	// [
    	//    { status: 'fulfilled', value: 42 },
    	//    { status: 'rejected', reason: -1 }
    	// ]
    ```

### Promise.any()

    - Promise.any()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。
    - Promise.any()跟Promise.race()方法很像，只有一点不同，就是不会因为某个 Promise 变成rejected状态而结束。

## Iterator

    - 遍历器（Iterator）就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。
    - 遍历是通过穿件一个指针对象，从数据结构的起始位置向后移动，是一种线性访问结构。每一次next移动指针返回的结果是一个{value:'value',done:Boolean}

### for...of

    - for...of默认就是调用的是数据结构的Symbol.iterator方法。如Array、Map、Set等都内置有iterator。
    - 循环得到的是集合的值。并且只能够返回数字索引的值，无法获取(字母)属性名的值。
    - 由于对象未实现有iterator接口，所以for...of不能够遍历对象实例。
    - for...of 不会遍历手动添加的其他键。
    - 能够按照顺序遍历数组。

### for...in

    - for...in循环读取的是集合的key值，能够遍历对象的键名，
    - 数组的键名是数字，但是for...in循环是以字符串作为键名。
    - for...in 循环不仅会遍历数字键名，还会遍历手动添加的其他键，甚至包含原型链上的键。
    - 某些情况下，for...in循环会以任意顺序遍历键名。
    - for...in主要是为遍历对象而设计，不适用于遍历数组。

### forEach

    - 无法中途跳出forEach循环，break命令或return命令都不能奏效。

## Generator 函数的语法

    - Generator函数从语法上理解，可以认为Generator函数是一个状态机，封装了多个内部状态。
    - 执行Generator函数会返回一个遍历器对象。这就意味着Generator函数也是一个遍历器对象生成函数。能够依次遍历Generator函数内部的每个状态。
    - 语法形式上，Ganerator函数在声明时function关键字与函数名之间有一个*号。在函数体内部通过使用yield表达式，定义不同的内部状态。
    - Generator函数的调用方式与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象（Iterator Object）。后续，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式或return语句为止。换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。
    - 每次遍历返回值形式为 { value: ‘value', done: true }。当执行到return语句时，正常返回表达式的值，done属性值为true，表示已经遍历结束。若继续执行则值为{ value: 'undefined', done: true }。后续再次调用next方法，返回的都是这个值。

### yield表达式

    - 由于 Generator 函数返回的遍历器对象，只有调用next方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。yield表达式就是暂停标志。
    - 遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。
    - 下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。
    - 如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。
    - 如果该函数没有return语句，则返回的对象的value属性值为undefined。
    - yield表达式只能够直接使用在Generator函数内部。（如Generator函数内部包裹普通函数，普通函数内部不能够使用yield）
    - 另外，yield表达式如果用在另一个表达式之中，必须放在圆括号里面。
    - yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号。
    ```javaScript
    	console.log('Hello' + yield 123); // SyntaxError
    	console.log('Hello' + (yield 123)); // OK
    	
    	function* demo() {
    	  foo(yield 'a', yield 'b'); // OK
    	  let input = yield; // OK
    	}
    ```
    - yield*表达式用于在一个Generator函数中调用另一个Generator函数。 

### for...of

    - 使用for...of遍历Generator函数并不会获取return 后的值，由于return的done属性为true则结束循环。
    ```javaScript
    	function* foo() {
    	  yield 1;
    	  yield 2;
    	  return 3;
    	}
    	for (let v of foo()) {
    	  console.log(v);
    	}
    	// 1 2 3 
    ```

### next方法的参数

    - yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。
    - 由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的。
    - 执行next()，函数只会执行到yield关键字，再次next()由断点保存处继续执行。
    - 使用for...of语句时不需要使用next方法。
    - 若next(value)带参数执行，则上次yield结果为本次传递参数。
    ```javaScript
    	function* foo(x) {
    	  var y = 2 * (yield (x + 1));
    	  var z = yield (y / 3);
    	  return (x + y + z);
    	}
    	
    	var a = foo(5);
    	a.next() // Object{value:6, done:false}  首次next()传递参数无效，x为声明Generator函数时传递的5。函数运行5+1
    	a.next() // Object{value:NaN, done:false} 未传递参数，首次yield结果为undefined，函数呈现为2*undefind = NaN； (NaN/3);
    	a.next() // Object{value:NaN, done:true} z = undefined; 5+NaN+undefined;
    	
    	var b = foo(5);
    	b.next() // { value:6, done:false }
    	b.next(12) // { value:8, done:false } 不考虑首次yield运算结果，由于next(12)故(yield (x + 1))=12; y=2*12;
    	b.next(13) // { value:42, done:true } 同理 z=13; 5+24+13
    ```

### Generator函数与Iterator接口的关系

    - Ganerator函数就是遍历器生成函数，因此可以吧Generator赋值给对象的Symbol.iterator属性，从而使该对象具有Iterator接口。
    - Generator函数执行后，返回一个遍历器对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。
    ```javaScript
    	var myIterable = {};
    	myIterable[Symbol.iterator] = function* () {
    	  yield 1;
    	  yield 2;
    	  yield 3;
    	};
    	[...myIterable] // [1, 2, 3]
    	
    	
    	function* gen(){
    	  // some code
    	}
    	var g = gen();
    	g[Symbol.iterator]() === g
    	// true
    ```
    - 对象原生并不具备Iterator接口，无法使用for...of进行遍历。但通过Ganerator可以通过添加接口或是将Generator添加到对象的Symbol.iterator属性上。
    ```javaScript
    //添加接口
    	function* objectEntries(obj) {
    	  let propKeys = Reflect.ownKeys(obj);
    	  for (let propKey of propKeys) {
    		yield [propKey, obj[propKey]];
    	  }
    	}
    	let jane = { first: 'Jane', last: 'Doe' };
    	for (let [key, value] of objectEntries(jane)) {
    	  console.log(`${key}: ${value}`);
    	}
    	
    //添加属性
    	function* objectEntries() {
    	  let propKeys = Object.keys(this);
    	  for (let propKey of propKeys) {
    		yield [propKey, this[propKey]];
    	  }
    	}
    	let jane = { first: 'Jane', last: 'Doe' };
    	jane[Symbol.iterator] = objectEntries;
    	for (let [key, value] of jane) {
    	  console.log(`${key}: ${value}`);
    	}
    ```

### Generator.prototype.throw()

    - Generator遍历器对象的throw方法和全局throw命令并不相同，前者由遍历器对象内部抛出，而后者则被函数体外的catch语句捕获。两者互不影响。
    - 由遍历器throw方法抛出的异常优先由遍历器进行捕获，若Generator函数内部没有做异常处理，则由函数外部异常处理捕获，若都未进行异常处理，则程序报错，中断执行。
    - 在 JavaScript 中，一次只能捕获一个异常的原因是异常处理是同步的。当 try...catch 块内捕获了一个异常后，这个块的控制流将从异常抛出的地方继续执行，而不会回到抛出异常的地方。所以catch()只能够捕获一次异常。
    - throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法。next方法一次都没有执行过。抛出的错误不会被内部捕获，而是直接在外部抛出，导致程序出错。因为第一次执行next方法，等同于启动执行 Generator 函数的内部代码，否则 Generator 函数还没有开始执行，这时throw方法抛错只可能抛出在函数外部。
    - throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。
    - Generator 函数体外抛出的错误，可以在函数体内捕获；反过来，Generator 函数体内抛出的错误，也可以被函数体外的catch捕获。

### generator.prototype.return()

    - return()，可以返回给定的值，并且终结遍历 Generator 函数。
    - 如果 Generator 函数内部有try...finally代码块，且正在执行try代码块，那么return方法会导致立刻进入finally代码块，执行完以后，整个函数才会结束。return的值会最后返回。
    - next()、throw()、return() 的共同点：这三个方法本质上作用都是让 Generator 函数恢复执行，并且使用不同的语句替换yield表达式。
    - next()是将yield表达式替换成一个值。
    - throw()是将yield表达式替换成一个throw语句。
    - return()是将yield表达式替换成一个return语句。
    ```javaScript
    	function* numbers () {
    	  yield 1;
    	  try {
    		yield 2;
    		yield 3;
    	  } finally {
    		yield 4;
    		yield 5;
    	  }
    	  yield 6;
    	}
    	var g = numbers();
    	g.next() // { value: 1, done: false }
    	g.next() // { value: 2, done: false }
    	g.return(7) // { value: 4, done: false }
    	g.next() // { value: 5, done: false }
    	g.next() // { value: 7, done: true }
    	
    	
    	const g = function* (x, y) {
    	  let result = yield x + y;
    	  return result;
    	};
    	const gen = g(1, 2);
    	gen.next(); // Object {value: 3, done: false}
    	gen.next(1); // Object {value: 1, done: true}
    	// 相当于将 let result = yield x + y
    	// 替换成 let result = 1;
    	
    	gen.throw(new Error('出错了')); // Uncaught Error: 出错了
    	// 相当于将 let result = yield x + y
    	// 替换成 let result = throw(new Error('出错了'));

    	gen.return(2); // Object {value: 2, done: true}
    	// 相当于将 let result = yield x + y
    	// 替换成 let result = return 2;
    ```

## async函数

    - async函数就是Generator函数的语法糖。简化异步操作。
    - 使用Generator函数的执行必须要依赖执行器，而async函数自带执行器。
    - 声明Generator的表示*号，以及yield，语义上与async以及await相同。
    - co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
    - 返回值是Promise，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。
    - async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误。也就是说，只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数。
    - async标记的函数只有在函数内部而言是继发的，外部不受影响，可并发执行。
    ```javaScript
    	async function logInOrder(urls) {
    	  // 并发读取远程URL
    	  const textPromises = urls.map(async url => {
    		const response = await fetch(url);
    		return response.text();
    	  });
    	  /**
    	   *await关键字确保代码等待异步操作完成，因此在自循环中的每个await都会等待对应的异步操作的完成，然后按次
    	   *序进行输出。
    	   */
    	  // 按次序输出 
    	  for (const textPromise of textPromises) {
    		console.log(await textPromise);
    	  }
    	}
    ```

### await

    - await 只能够用在async函数中。
    - 正常情况下，await命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。
    - await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到。reject方法无需return，仍然可以传入catch方法的回调函数。
    - 任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行。若希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以将第一个await放在try...catch结构里面，这样不管这个异步操作是否成功，第二个await都会执行。另一种方法是await后面的 Promise 对象再跟一个catch方法，处理前面可能出现的错误。
    ```javaScript
    	async function f() {
    	  try {
    		await Promise.reject('出错了');
    	  } catch(e) {
    	  }
    	  return await Promise.resolve('hello world');
    	}

    	async function f() {
    	  await Promise.reject('出错了')
    		.catch(e => console.log(e));
    	  return await Promise.resolve('hello world');
    	}
    ```
    - 如果将forEach方法的参数改成async函数，代码可能不会正常工作，原因是forEach内的操作将是并发执行，也就是同时执行，而不是继发执行。正确的写法是采用for循环或是reduce。
    - 若希望多个请求并发执行，可以使用Promise.all方法。

### 顶层await（没有使用async标记的函数进行包裹的await）

    - 默认页面加载是按顺序加载，同步运行。页面每个操作按照顺序依次进行触发，但并不关心每步操作是否完成。
    - 异步操作async，await标记过的后，若标记的语句未执行完成，则不执行后续代码。

## Module

### 浏览器加载

    - 页面加载javaScript脚本，下面两种方式都是异步加载，在浏览器加载时，并不影响页面渲染。
    ```javaScript
    <script src="path/to/myModule.js" defer></script>	//defer是“渲染完再执行”
    <script src="path/to/myModule.js" async></script>	//是“下载完就执行”
    ```

### ES6模块 与 CommonJs 模块的差异

    - CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
    - CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
    - CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。
    - ES6模块遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的import有点像 Unix 系统的“符号连接”，原始值变了，import加载的值也会跟着变。
    ```javaScript
    //CommonJs 
    // lib.js
    var counter = 3;
    function incCounter() {
      counter++;
    }
    module.exports = {
      counter: counter,
      incCounter: incCounter,
    };
    // lib.js模块加载以后，它的内部变化就影响不到输出的mod.counter了。这是因为mod.counter是一个原始类型的值，会被缓存。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。也就是说mod.counter和lib.js模块内的counter并没有关联关系，incCounter()影响的是内部counter，而不是export导出的拷贝。
    var mod = require('./lib');
    console.log(mod.counter);  // 3
    mod.incCounter();
    console.log(mod.counter); // 3

    // lib.js 除非写成一个函数，才能得到内部变动后的值。
    var counter = 3;
    function incCounter() {
      counter++;
    }
    module.exports = {
      get counter() {
    	return counter
      },
      incCounter: incCounter,
    };

    ```

    ```javaScript
    // ES6模块  不同的脚本加载这个模块，得到的都是同一个实例。
    // mod.js
    function C() {
      this.sum = 0;
      this.add = function () {
    	this.sum += 1;
      };
      this.show = function () {
    	console.log(this.sum);
      };
    }
    export let c = new C();


    // x.js
    import {c} from './mod';
    c.add();

    // y.js
    import {c} from './mod';
    c.show();

    // main.js
    import './x';
    import './y';

    $ babel-node main.js
    1
    ```

## Hooks

### useEffect

    - 根据Effect的声明，可以看到useEffect的第一个参数是effect的回调，第二个参数是deps依赖项（可选、数组类型）。useEffect会根据依赖项决定是否调用EffecCallback。
    ```javaScript
    	type DependencyList = ReadonlyArray<any>;
    	function useEffect(effect: EffectCallback, deps?: DependencyList):void;
    ```
    - useEffect调用时机
    	1. 与页面渲染的先后关系：React会等待浏览器完成画面渲染之后才会延迟调用useEffect。
    	2. 调用次数：在不传递deps依赖项的情况下，useEffect会在每次渲染后都执行。若传递空数组，则只在页面第一次渲染后执行一次。若传递参数为非空数组，则在每次页面渲染后，以及非空数组内包含的state参数发生更新时，就会触发EffectCallback，即useEffect传入回调函数。

    - useEffect 不同依赖项相关影响
    	- 不传递依赖项
    		1. 调用时机：在不传递deps依赖项时，useEffect会在每次渲染后都执行，即它会在第一次渲染之后和每次更新之后都会执行。相当于在 componentDidMont 和 componentDidUpdate 两个类方法中执行。
    		2. 适用场景：当effect回调计算量小，不会触发页面渲染的情况。
    ```javaScript
    	useEffect(() => {
    		回调;
    	});
    ```
    	- 传递空数组作为依赖项
    		1. 调用时机：只会在第一次页面渲染之后执行回调，相当于 componentDidMont。
    		2. 适用场景：希望只在页面第一次渲染时执行，不需要每次页面渲染后都执行。
    ```javaScript
    	// query 
    	const [comments,setComments] = useState()
    	useEffect(() => {
    		fetchComments().then(response => {
    			setComments(response.data)});
    		},[]);
    ```
    	- 传递非空数组作为依赖项
    		1. 调用时机：当非空数组内的state参数有更新时，就会执行回调函数。
    		2. 适用场景：需要针对特定参数对页面进行再渲染。
    		3. 值比较：
    			当依赖项为非空数组时，则在每次渲染后，非空数组上的state发生更新时，就会执行回调。
    			Effect会对依赖项进行浅比较来确定依赖项是否发生更新。
    			JavaScript 中数据类型分为 值类型和引用类型。
    			值类型：浅比较是对变量值进行比较。
    			引用类型：浅比较是对对象指向的内存地址进行比较。

    - useEffect 清除机制
    	1. useEffect 回调函数EffectCallback的返回值 void|Destructor，其中Destructor是可选的析构函数。
    ```javaScript
    	type EffectCallback = () => (void | Destructor);
    ```
    	2. 当依赖项更新时，会先调用上一次渲染后执行的EffectCallback的销毁函数。
    ```javaScript
    	function Counter(seconds){
    		const [cont,setCount] = useState(0);
    		
    		useEffect(() => {
    			const id = setInterval(() => {
    				setCount( c=>c+1);
    			}, 1);
    			// 返回销毁
    			return ()= > clearInterval(id);
    		},[seconds]);
    		
    		return {count};
    	}
    ```

