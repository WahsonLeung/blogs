# ES6初探（一）

标签（空格分隔）： 前端技术

---
[TOC]

### 1、块级绑定
- **var声明和变量提升**：使用var声明变量时，无论其实际声明位置在何处，都会被视为声明与所在函数的顶部，如果声明不在任意函数内，则视为在全局作用域的顶部。
- **块级声明**：让所声明的变量在指定块的作用域外无法被访问。
- **let 声明**：let声明的语法和var一样，但是let声明的变量不会提升到代码块的顶部，且变量的作用域会被限制在当前代码块中。
    - 在同一个代码块内，let不能声明已经被定义过的标识符。
- **const 声明**：常量声明。与let声明一样是块级声明。语法同let和var。
    - 在同一个代码块内，const不能声明已经被定义过的标识符。这点与let相似。
    - 当使用const声明对象时，const声明会阻止变量绑定到一个新的对象，但并不阻止绑定对象中成员值得改变。
- **循环内使用块级声明**
    - 循环内的let声明：在循环的每次迭代中，都会创建一个新的同名变量并对其进行初始化。考虑以下的情况，代码的意图是依次输出0到9
        - ES5时代：

        ```javascript
        var funcs = [];
        for(var i = 0; i < 10; i++) {
            funcs.push(function(){console.log(i)});
        }
        funcs.forEach(function(fn) {
            fn();//实际上，结果是输出了10个9
        });
        
        //---------------分界线--------------------
        //可以使用立即调用表达式来修正上面的问题
        for(var i = 0;i < 10; i++){
            funcs.push(function(value){
                return function(){
                    console.log(value);
                }
            }(i));
        }
        funcs.forEach(function(fn){
            fn();//从 0 到 9 依次输出
        });
        ```
        - ES6时代:
        
        ```javascript
        const funcs = [];
        for(let i = 0; i < 10; i++){
            funcs.push(() => console.log(i));
        }
        funcs.forEach(fn => fn());//从 0 到 9 依次输出
        ```
    - 循环内的const声明
        - 在fori循环中使用const声明，i被声明为常量，循环的第一次迭代成功执行，但是第二次的时候会报错，因为这里试图改变常量i的值。
        - 要注意const用于for-in或for-of循环中时，效果与let相同。有别于fori循环，for-in和for-of循环为每次迭代创建一个新的变量绑定，而不是试图去修改已绑定的变量的值。
- **全局作用域**

    ```javascript
    //RegExp是一个全局属性，它定义在window上的，当使用var时可以轻易把它覆盖掉：
    var RegExp = "hello";
    console.log(window.RegExp); // "hello"
    
    //-----------------分割线---------------------
    //然而，在全局作用域上使用let或const时，虽然在全局作用域上创建了新的绑定，但是不会有任何属性被添加到全局对象上，也就是不能覆盖一个全局变量，而只是把它屏蔽了。
    let RegExp = "hello";
    console.log(RegExp); // "hello"
    console.log(window.RegExp === RegExp); //false
    ```
### 2、字符串和正则表达式
- **String.prototype.repeat()**
    - ES6为字符串添加了一个repeat()方法，repeat()接收一个参数作为字符串的重复次数。
    ```
    console.log("6".repeat(6)); // "666666"
    console.log("abc".repeat(2)); // "abcabc"
    ```
    
- **正则表达式的粘连特性**
    - 关于粘连标志的微妙细节：
        - 只有调用正在表达式对象上的方法（如text()与exec()方法），lastIndex属性才会生效。而将正在表达式作为参数传递给字符串上的方法（如match()），并不会体现粘连特性。

- **模板字面量**
    - 标签化模板：模板字面量真正的魔力来源于标签化模板，一个模板标签（template tag）能对模板字面量进行 *转换* 并返回最终的字符串值。用法：``let msg = tag`helloworld`。 ``一个标签（tag）仅是一个函数，它被调用时接收需要处理的模板字面量数据。
    &nbsp;
    看下面的例子：
    ```javascript
    function raw(literals,...subsitutions){
    	let result = '';
    	for(let i = 0,len = subsitutions.length; i < len; i++){
    		result += literals.raw[i];
    		result += subsitutions[i];
    	}
    	result += literals.raw[literals.length - 1];
    	return result;
    }
    // 
    let msg1 = 'message1', msg2 = 'message2';
    raw`${msg1}mutil\n${msg2}`      // "message1mutil\nmessage2"
    ```
        这段代码的意图是把字面量元素（literals）的值转变为原始值并输出。
    参数``literals``是一个包含额外属性``raw``（raw属性是由每个字面量值的原始值组成的数组）的数组，其结构如下：
    <span style="margin-left:10px;"></span>![此处输入图片的描述][1]
    参数``subsitutions``也是一个数组，包含了替换符的计算值：
    <span style="margin-left:10px;"></span>![此处输入图片的描述][2]
    
### 3、函数
- **参数默认值**
    - 用法
    ```javascript
    //1. 当p2、p3、p4参数未传递或者直接传undefined时，他们会使用默认值
    function func(p1, p2 = 2, p3 = {}, p4 = () => {}, p5) {...}
    
    //2. 参数默认表达式：使用函数的返回值作为参数默认值
    function getDefParam(value){
        return value + 1;
    }
    
    // p2把getDefParam()执行后的返回值作为默认值，p1作为参数传递给了getDefParam()
    function func1(p1, p2 = getDefParam(p1)) {...}
    
    //3. 把前面的参数作为后面参数的默认值
    function func2(p1, p2 = p1) {...}
    ```
- **arguments对象的变化**
    - ES5: 在非严格模式下，具名参数的变化会反映到arguments对象上，而在严格模式下，则不会。  
    ```javascript
    function mixArgs(first,second) { 
        console.log(first === arguments[0]);// true
        console.log(second === arguments[1]); // true
        first = "c";
        second = "d";
        console.log(first === arguments[0]); //非严格模式：true，严格模式：false
        console.log(second === arguments[1]);//非严格模式：true，严格模式：false
    }
    mixArgs("a","b");
    ```
    
    - ES6：在使用*参数默认值*时，arguments对象的表现与ES5的严格模式一致。
    ```javascript
    function mixArgs(first,second="b") {
        console.log(arguments.length);  // 1 
        console.log(first === arguments[0]);// true
        console.log(second === arguments[1]); // false
        first = "c";
        second = "d";
        console.log(first === arguments[0]); //false
        console.log(second === arguments[1]);//false
    }
    mixArgs("a");
    ```
        mixArgs传递了一个参数，所以arguments\[1\]的值是undefined，改变first和second的值不会对arguments对象造成影响，因此可以使用arguments对象来反映函数的初始调用状态。
- **剩余参数**（rest parameter）
    - 设计剩余函数是为了替代ES中的arguments。因此需要使用arguments的地方都可以使用剩余参数替代。
    - 用法：
    ```javascript
    function func1(...args) {
        ...
    }
    function func2(object, ...args) {
        ...
    }
    ```
    
    - 限制：
        1. 函数只能有一个剩余参数，并且必须被放在最后。
        2. 剩余参数不能在对象字面量的setter属性中使用。
    
- **更强的函数构造器**
    - 函数构造器中同样可以使用参数默认值和剩余参数。
    ```javascript
    var add = new Function("first", "second = first", "return first + second;");
    console.log(add(2)); // 4
        
    var pickFirst = new Function("...args", "return args[0];");
    console.log(pickFirst(1, 2)); // 1
    ```
- **扩展运算符**
    - 我们已经对剩余参数有所了解，它允许你把多个独立的参数合并到一个数组中；而扩展运算符则是把一个数组分割，并将各项作为分离的参数传给函数。    
        
        ```javascript
        let values = [1, 2, 3, 4, 5];
        // Math.max允许接受多个参数值进行比较，但不能接受数组，使用扩展运算符等同于Math.max(1, 2, 3, 4, 5);
        Math.max(...values);
        
        // 扩展运算符可以与其他参数混用
        Math.max(...values, 6, 7);
        ``` 
- **函数的name属性**
    - ES6中所有函数都有适当的name属性值。
    ```javascript
    // 1.
    function doSth(){};
    console.log(doSth.name); // "doSth"
    
    // 2.
    var doAnotherThing = function(){};
    console.log(doAnotherThing.name); // "doAnotherThing"
    
    // 3. 函数表达式自己的名称优先级要高于赋值目标的变量名
    var doSth = function doSthElse(){};
    console.log(doSth.name); // "doSthElse"
    
    let person = {
        get firstName(){
            return "Wahson";
        },
        sayName:function(){
            console.log(this.name);
        }
    };
    console.log(person.sayName.name); // "sayName"
    
    // 4. firstName实际上是一个getter函数
    let descriptor = Object.getOwnPropertyDescriptor(person, "firstName");
    console.log(descriptor.get.name); // "get firstName"
    
    // 5. 绑定产生的函数拥有原函数的名称，并总会附带"bound"前缀
    console.log(doSth.bind().name); // "bounddoSth"
    
    // 6. 使用Function构造器创建的函数，名称属性值是"anonymous"
    console.log(new Function().name); // "anonymous"
    ```
- **函数的双重用途**
    - 在ES5中，根据是否使用new，函数会有不同的用途。
    ```javascript
    function Person(name) {
        this.name = name;
    }
    
    // 1. 使用new时，函数作为构造函数，内部的this是一个新对象，并作为函数的返回值。
    let person = new Person("Wahson");
    console.log(person); // [Object object]
      
    // 2. 当未使用new时，Person("Wahson")，作为一般的函数调用，返回了undefined
    // 且非严格模式下，this指向全局对象，故全局对象会被添加name属性
    let notAPerson = Person("Wahson");
    console.log(notAPerson); // "undefined"
    ```javascript
        JS提供了两个内部方法``[[Call]]``和``[[Construct]]``，当函数使用new调用是，[[Construct]]会被执行，负责创建一个新的对象，并使用该对象作为this去执行函数体；当未使用new时，[[Call]]方法执行，执行的是代码中显示的函数体。
        
    - ES5中判断是否使用new调用函数的方法
    ```javascript
    function Person(name){
        if(this instanceof Person) {
            this.name = name;
        } else {
            throw new Error("You must use new with Person");
        }
    }
    
    var person = new Person("Wahson");
    
    // 但是这种方法可以通过call或apply方法改变this指向，因此下面的代码可以成功调用Person()
    var notAPerson = Person.call(person,"Leung");
    ```
    - ``new.target``元属性：ES6引入了new.target元属性（指非对象如new，上的属性）。当函数的[[Construct]]被调用，new.target会被填入新创建的对象实例的构造器，当[[Call]]被调用，new.target会是undefined。
    ```javascript
    function Fn(){
        console.log(new.target === Fn);  // true
    }
    
    new Fn(); 
    ```
- **块级函数**
    - 先看代码
    ```javascript
    "use strict";
    if(true) {
        // ES5中，代码块内部的函数声明会抛出语法错误，而ES6允许这种情况
        function doSth() {}
    }
    ```
    
    - ES6将doSth()视为块级声明，并允许它在定义的代码块内部被访问。
    ```javascript
    "use strict";
    if(true) {
        // 块级函数会被提升到所在代码块的顶部
        console.log(typeof doSth); // "function"
        function doSth() {}
        doSth();
        
        try {
            console.log(typeof doSth1); // 抛出错误
        } catch (e) {
            console.error(e);
        }
        // let 函数表达式
        let doSth1 = function(){};
        doSth1();
    }
    // 与let函数表达式相似，在执行流跳出定义所在的代码块后，函数定义会被移除。
    console.log(typeof doSth); // "undefined"
    console.log(typeof doSth1); // "undefined"
    ```
        要注意一点：ES6非严格模式下，块级函数的作用域会被提升到所在函数或者全局环境的顶部，而不是代码块的顶部。
- **箭头函数**
    - 用法
    ```javascript
    // 1. 只有一个参数
    // 等价于 
    // let reflect = function(value) {return value;}
    let reflect = value => value;
    
    // 2. 多个参数
    // 等价于 
    // let reflect = function(num1, num2) {return num1 + num2;}
    let sum = (num1, num2) => num1 + num2;
    
    // 3. 没有参数
    // 等价于
    // let getName = function() {return "Wahson";};
    let getName = () => "Wahson";
    
    // 4. 返回一个对象字面量，需要使用圆括号包裹
    // 等价于
    // let getTempItem = function(id) {return {id: id, name: "Temp"}};
    let getTempItem = id => ({id: id, name: "Temp"});
    
    // 5. 立即调用表达式（IIFE）
    let person = (name => ({
        getName() {
            return name;
        }
    }))("Wahson");
    console.log(person.getName()); // "Wahson"
    ```
    - 没有this绑定：箭头函数内部的this只能通过查找作用域链来确定，如果箭头函数包含在一个非箭头函数内，那么this值就会与 _该函数的_ 相等，否则this就会是undefined。由于this值由包含它的函数决定，因此 _使用call()、apply()或bind()方法并不能改变其this值_。
    - 箭头函数缺失prototype属性，它不能用于定义新的类型。
    ```javascript
    let MyType = () => {};
    // 错误！！！不能对箭头函数使用new
    let obj = new MyType(); 
    ```
    - 没有arguments 绑定：箭头函数没有自己的arguments对象，但是仍然能够访问包含它的函数的arguments对象。

- **尾调用优化**
    - 尾调用指的是调用函数的语句是另一个函数的最后语句，就像这样
    ```javascript
    function doSth() {
        return doSthElse(); // 尾调用
    }
    // 在ES5引擎中，尾调用跟其他的函数调用一样：一个新的栈帧（stackframe）被创建并推到调用栈顶部。
    // 之前的每个栈帧都会被保留在内存中，当调用栈太大时，就有可能导致堆栈溢出。
    ```
    - ES6在严格模式下会试图进行尾调用优化，来减少调用栈的大小。（非严格模式下，不会进行优化）_满足以下条件时，尾调用会清楚当前栈帧并再次利用它，而不是创建一个新的栈帧_。
        - 尾调用没有没有引用当前栈帧中的变量（意味着该函数不能是闭包）。
        - 进行尾调用的函数在尾调用返回结果后不能做额外的操作。
        - 尾调用的结果作为当前函数的返回值。
    
        ```javascript
        "use strict";
        function doSth () {
            // 1. 未被优化，缺少return
            doSthElse();
        }
        
        function doSth () {
            // 2. 未被优化，未调用返回后还执行加法
            return 1 + doSthElse();
        }
        
        function doSth () {
            // 3. 未被优化，调用不在尾部
            let result = doSthElse();
            return result;
        }
        
        function doSth () {
            let num = 1, func = () => num;
            // 4. 未被优化，func是闭包
            return func();
        }
        ```
### 4、扩展的对象功能
- **属性初始化器的速记法**
    - 在ES6之前，对象字面量是 _键值对_ 的简单集合，在属性初始化的时候如果属性名和参数名（变量名）相同，就会出现下面这种重复的情况。
    ```javascript
    function createPerson(name, age) {
        return {
            name: name,
            age: age
        };
    }
    ```
    
    - ES6使用 _属性初始化器的速记法_ 可以把上面的代码简写为：
    ```javascript
    function createPerson(name, age) {
        return {name, age};
    }
    ```
    
- **方法简写**
    - ES5为对象添加方法，需要指定一个名称并用完整的函数定义。
    ```javascript
    var person = {
        name: "Wahson",
        sayName: function() {
            console.log(this.name);
        }
    }
    ```
    - ES6通过省略冒号与``function``关键字简写了方法的定义。
    ```javascript
    var person = {
        name: "Wahson",
        sayName() {
            console.log(this.name);
        }
    }
    ```

- **需计算属性名**
    - 在ES6之前，要获得对象的一个属性的值，需要事先知道属性的名称，然后通过方括号表示法（ ``obj[propName]`` ）或者小数点表示法来获取（ ``obj.propName`` ）。但是 ``propName`` 不能通过计算获得。而ES6支持了通过计算获得属性名。
    - 用法 
    ```javascript
    const suffix = " name";
    var person = {
        ["first" + name] : "Wahson",
        ["last" + name] : "Leung"
    }
    
    console.log(person["first name"]); // "Wahson"
    console.log(person["last name"]);  // "Leung"
    ```

- **Object.is()**
    - Object.is()方法严格相等运算符（ ``===`` ）的升级版。在大部分情况下，Object.is()的结果与 ``===`` 运算符是相同的，仅有的例外是：
    ```javascript
    // 1. +0 -0 
    console.log(+0 == -0);     // true
    console.log(+0 === -0);    // true
    console.log(Object.is(+0,-0));   //false
    
    // 2. NaN
    console.log(NaN == NaN);    //false
    console.log(NaN === NaN);   // false
    console.log(Object.is(NaN,NaN));   // true
    
    ```
        但是，仍然没有必要停止使用``===``，选择使用 ``Object.is()`` 还是 ``==`` 或 ``===`` ，应该取决与代码的实际情况。

- **Object.assign()**
    - ES6中标准化的 _拷贝对象属性_ 的方法（注意：是浅拷贝）。该方法接受一个接收者，以及任意数量的供应者，并返回接收者。接收者会按照供应者在参数中的顺序来依次接收它们的属性。也就是说，后面的供应者的属性可能会覆盖前面供应者的。
    ```javascript
    let receiver = {};
    Object.assign(receiver,{
            type: "js",
            name: "file.js"
        },
        {
            type: "css"
        }
    );
    console.log(receiver.type);  // "css"
    console.log(receiver.name);  // "file.js"
    ```
    - 另外，需要注意的是，Object.assign() 无法复制 _访问器属性（getter）_。
    ```javascript
    let receiver = {};
    let supplier = {
        get name() {
            return "Wahson"
        }
    };
    
    Object.assign(receiver,supplier);
    let descriptor = Object.getOwnPropertyDescriptor(receiver, "name");
    
    console.log(descriptor.get);    // undefined
    console.log(descriptor.value);  // "Wahson"
    console.log(receiver.name);     // "Wahson"
    ```
        - 来看一下``supplier``对象的结构：
        ![此处输入图片的描述][3]
        
        - ``receiver``对象的结构：
        ![此处输入图片的描述][4]

- **重复对象字面量属性**
    - ES5严格模式下，不允许重复的属性名。在ES6中，严格模式和非严格模式都不再检查重复的属性，当存在重复属性，排在后面的属性的值会成为该属性的实际值。
    ```javascript
    "use strict";
    let person = {
        name: "Wahson",
        name: "Handsome"  // ES5严格模式下是语法错误
    }
    console.log(person.name);    // "Handsome"
    ```
    
- **自有属性的枚举顺序**
    - ES6中严格定义了自由属性在被枚举是返回的顺序。
        - 所有的数字类型键，按升序排列。
        - 所有的字符串类型键，按照被添加到对象的顺序排列。
        - 所有的符号类型键，也按添加顺序排列。
    
        ```javascript
    let obj = {a:1, 0:1, b:1, 2:1, c:1, 1:1 };
    obj.d = 1;
    console.log(Object.getOwnPropertyNames(obj).join(""));  // 012abcd
    ```
        
- **Object.setPrototypeOf()**
    - 在ES5，对象通过构造器或者 ``Object.create()`` 创建后，原型就会被指定，而之后 _不能再被改变_。ES6添加了 ``Object.setPrototypeOf()`` 方法来修改对象的原型。
    ```javascript
    let person = {
        getGreeting() {
            return "Hello";
        }
    }
    
    let dog = {
        getGreeting() {
            return "Woof";
        }
    }
    
    // 原型为 person
    let friend = Object.create(person);
    console.log(friend.getGreeting());                      // "Hello"
    console.log(Object.getPrototypeOf(friend) === person);  // true
    
    // 将原型修改为 dog
    Object.setPrototypeOf(friend,dog);
    console.log(friend.getGreeting());                      // "Woof"
    console.log(Object.getPrototypeOf(friend) === dog);  // true
    ```
- **简写方法内的super引用**
    - ES6引入了super，super是指向当前对象的原型的一个指针，因此可以通过super调用对象原型上的任何方法。但是 **super必须位于简写方法之内** ，否则会导致语法错误。
    ```javascript
    let person = {
        getGreeting() {
            return "Hello";
        }
    }
    let friend = {
        getGreeting() {
            return super.getGreeting() + ", Hi!";
        }
    }
    
    Object.setPrototypeOf(friend, person);
    
    let relative = Object.create(friend);
    console.log(person.getGreeting());  // "Hello"
    console.log(friend.getGreeting()); //  "Hello,Hi!"
    console.log(relative.getGreeting()); //  "Hello,Hi!"
    ```
- **ES6名正言顺的“方法”定义**
    - ES6之前并没有正式定义“方法”这个概念，它此前仅指对象的函数属性。ES6中正式定义了``方法``：一个拥有``[[HomeObject]]``内部属性的函数，此函数属性指向该方法所属的对象。
    ```javascript
    let person = {
        // 方法
        getGreeting() {
            console.log("Hello");
        };
    };
    
    // 并非方法
    function shareGreeting() {
        return "Hi";
    }
    ```
        方法和普通的函数的区别在于是否具有内部属性``[[HomeObject]]``，这一点在使用super调用原型上的方法时至关重要。任何对super的引用都会在 ``[[HomeObject]]`` 上调用 ``Object.getPrototypeOf()`` 来获取原型，然后在原型上查找同名的函数，最后创建 ``this`` 绑定并调用该方法。

  [1]: http://om0gfsbon.bkt.clouddn.com/20170601149632458088770.png
  [2]: http://om0gfsbon.bkt.clouddn.com/20170601149632466172168.png
  [3]: http://om0gfsbon.bkt.clouddn.com/20170608149690711576836.png
  [4]: http://om0gfsbon.bkt.clouddn.com/20170608149690707594400.png