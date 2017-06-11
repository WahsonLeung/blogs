# ES6初探（二）

标签（空格分隔）： 前端技术

---
[TOC]

### 5、解构：更方便的数据访问
- **对象解构**
    - 直接看用法
    ```javascript
    // 1. 对象解构
    let node = { type: "id" , name: "foo"};
    let {type , name} = node;
    console.log(type); // "id"
    console.log(name); // "foo"
    
    
    // 2. 在赋值的时候使用解构
    let node = { type: "id" , name: "foo"},
        type = "literal",
        name = "bar";
    ({type, name} = node);  // 在声明之后改变他们的值
    console.log(type); // "id"
    console.log(name); // "foo"
    
    
    // 3. 解构传参两不误
    function output(value) {
        console.log(value === node); // true
    }
    output({type, name} = node);
    console.log(type); // "id"
    console.log(name); // "foo"
    
    
    // 4. 解构时指定默认值
    let {type, name, value = true} = node;
    console.log(value); // true，如果没有指定默认值，value将是undefined
    
    
    // 5. 把解构的值赋给不同的本地变量名
    let node = { type: "id"},
    let {type: localType, name: localName = "bar"} = node;
    console.log(localType);   // "id"
    console.log(localName);  // "bar"
    console.log(type);      // ReferenceError: type is not defined
    
    
    // 6. 嵌套对象的解构
    let node = {
        loc: {
            start: {
                line: 1, column : 1
            }
        }
    };
    // 提取node.loc.start
    let { loc: { start } } = node;
    console.log(start.line); // 1
    console.log(loc);    //  ReferenceError: loc is not defined
    
    
    // 7. 嵌套结构中指定本地变量名
    // 提取node.loc.start，并写入到本地变量
    let { loc: { start: localStart } } = node;
    console.log(localStart.line); // 1
    console.log(start);   // ReferenceError: start is not defined
    console.log(loc);    //  ReferenceError: loc is not defined
    
    
    // 8. 数组解构
    let colors = ["red", "green", "blue"];
    let [first, second] = colors;
    console.log(first);    // "red"
    console.log(second);   // "green"
    
    
    // 9. 忽略不需要的项
    let colors = ["red", "green", "blue"];
    let [,,third] = colors;
    console.log(third);   // "blue"
    
    
    // 10. 使用数组解构赋值对变量互换值
    let a = 1, b = 2;
    [a, b] = [b, a];  // 这里[b, a]创建了一个临时的数组字面量[1, 2]，然后对该数组进行解构
    console.log(a);   // 2
    console.log(b);   // 1
  
    
    // 11. 数组解构时指定默认值
    let colors = ["red"];
    let [first, second = "green"] = colors;
    console.log(first);     // "red"
    console.log(second);    // "green"
    
    
    // 12. 嵌套的数组解构
    let colors = ["red", ["green", "blue"]];
    let [first, [secord, third]] = colors;
    console.log(first);    // "red"
    console.log(secord);    // "green"
    console.log(third);    // "blue"
    
    
    // 13. 剩余项，使用...语法将剩余的项赋值给一个变量，剩余项必须是数组解构模式中最后的部分，之后不能再有逗号
    let colors = ["red", "green", "blue"];
    let [first, ...rest] = colors;   // 
    console.log(first);         // "red"
    console.log(rest.length);   // 2
    console.log(rest[0]);       // "green"
    
    
    // 14. 使用剩余项进行数组克隆
    let [...clonedColors] = colors;
    console.log(clonedColors);   // "[red, green, blue]"
    
    
    // 15. 混合解构
    let node = {
        loc: {
            start: {
                line: 1, column : 1
            }
        },
        range: [0, 1, 2, 3]
    };
    // 提取node.loc.start
    let { loc: { start }, range:[startInd] } = node;
    console.log(start.line); // 1
    console.log(startInd);   // 0
    
    
    // 16. 参数解构
    function setCookie(name, value, {secure, path, domain, expires}) {
        // ...
    }
    setCookie("type", "js", {secure:true, expires: 6000});
    // 这里，如果不给参数解构传值，会抛出错误
    setCookie("type", "js");  // 没有传第三个参数，抛错
    
    
    // 17. 提供默认值，使解构参数可选
    function setCookie(name, value, {secure, path, domain, expires} = {}) {
        // ...
    }
    
    
    // 18. 同样，解构时也可以提供默认值
    function setCookie(name, value, {
            secure = false, 
            path = "/", 
            domain = "example.com", 
            expires = new Date()
        } = {}) {
        // ...
    }
    
   
    ```
    - 有一点要注意，不管是对象还是数组解构，都必须提供初始化器（即等号右边的值）。
    ```javascript
    // 以下情况都会引起语法错误。
    var {type, name};  
    let {type, name};
    const {type, name};
    
    var [first,second];
    let [first,second];
    const [first,second];
    ```

### 6、符号与符号属性
- **新的基本类型：Symbol**
    - ES6引入了一种新的基本类型：符号（Symbol）。
    - 创建符号值
    ```javascript
    let firstName = Symbol("firstName");   // 也可以不传参数
    let person = {};
    person[firstName] = "Wahson";
    console.log(person[firstName]);  // "Wahson"
    console.log(firstName);  //   "Symbol(firstName)"
    
    ```
    - 共享符号值
    
### 7、Set与Map
- **Set**
    - ES6新增了Set类型，Set是一种无重复值的有序列表。值重复的比较在Set内部使用 ``Object.is()``。
    ```javascript
    let set = new Set();
    set.add("5");
    set.add(5);
    console.log(set.size);   // 2
    // 测试值是否存在
    console.log(set.has(5));  // true

    // 删除值
    set.delete(5); 
    
    // 清空集合
    set.clear();    
    ```
    - Set的遍历。Set是没有键的，但是为了使Set、Map、数组的forEach方法的回调函数保持参数相同，Set的每一项同时认定为键与值。
    ```
    // 遍历集合
    set.forEach((value, key, ownerSet) => {
        console.log(`${key}: ${value}`);   
        console.log(ownerSet === set); 
    });
    // 输出结果
    // 5: 5
    // true
    // 5: 5
    // true
    ```
    - Set与Array的相互转化
    ```
    // 使用数组初始化set
    let set = new Set([1,2,3,4,5,5,5]);
    console.log(set);   // Set {1,2,3,4,5}
    
    // 把set转成数组
    let arr = [...set];  // [1,2,3,4,5]
    ```
- **WeakSet**
    - 对象存储在Set中，实际上相当于把对象存储在变量中，只要Set实例的引用仍然存在，所存储的对象就无法被垃圾回收机制回收，从而无法释放内存。为了缓解这个问题，ES6增加了WeakSet类型。WeakSet只允许存储对象的弱引用，对象的弱引用在它自己成为该对象的唯一引用时，不会阻止垃圾回收。故而，_WeakSet只储存对象弱引用，它就不能储存基本类型的值_。
    ```javascript
    let set = new WeakSet(), key = {};
    set.add(key);
    // 测试值是否存在
    console.log(set.has(key));  // true
    
    key = null; // 此时弱引用成为了唯一的引用
    console.log(set.has(key));  // false
        
    ```
    - WeakSet与Set的最大区别是对象的弱引用。另外，对于WeakSet
        - 只要调用 ``add()``、``has()`` 或 ``delete()`` 方法时传入了非对象的参数，就会抛出错误。
        - 不可迭代，因此不能用在 ``for-in`` 循环中。
        - 无法暴露出任何迭代器（例如keys()与values()），因此没有任何编程的手段用于判断WeakSet的内容。
        - 没有 ``forEach()`` 方法。
        - 没有 ``size`` 属性。
- **Map**
    - ES6的Map类型是键值对的有序列表，而 _键和值都可以是任意类型_ 。键的比较使用的是 ``Object.is()``。
    ```javascript
    let map = new Map([["name", "Wahson"],["age", 25]]);
    map.set("job", "engineer");
    console.log(map.get("name"));   // "Wahson"
    console.log(map.get("age"));    // 25
    
    let key = {};
    map.set(key, 6);       // 将对象作为键
    
    console.log(map.size);  // 4
    // 检查指定的键时候存在
    console.log(map.has(key));   // true
    // 删除键值对
    map.delete("name");
    // 清空键值对
    map.clear();
    
    ```
    - Map的遍历。
    ```javascript
    map.forEach((value, key, ownerMap) => {
        console.log(`${key} => ${value}`);
        console.log(map === ownerMap);
    });
    // 输出结果
    // name => Wahson
    // true
    // age => 25
    // true
    // job => engineer
    // true
    // ...
    ```
        ``forEach()`` 的回调函数接收每个键值对的顺序，是按照键值对被添加到Map中的顺序。
- **WeakMap**
    - WeakMap之于Map，就像WeakSet之于Set。WeakMap中，所有的键都必须是对象的弱引用。当WeakMap中的键在WeakMap之外不存在引用时，改键值对会被移除。
    ```javascript
    let key1 = {}, 
        key2 = {}, 
        key3 = {},
        map = new WeakMap([[key1, "Hello"], [key2, 66]]);
    map.set(key3, {});
    console.log(map.has(key1));  // true
    console.log(map.get(key1));  // "Hello"
    // 删除键值对
    map.delete(key3); 
    
    key2 = null;  // 此时弱引用成为了唯一的引用
    console.log(map.has(key2));  // false
    
    ```
    - 同样，WeakMap没有 ``forEach``方法，``size``属性，甚至``clear()``方法。 
### 8、迭代器与生成器

### 9、JS的类
- **ES5中的仿类结构**
    - ES6之前都不存在类，与类最接近的是创建一个构造器，然后将方法指派到该构造器的原型上。这种方式通常被称为创建自定义类型。
    ```javascript
    function PersonType(name) {
        this.name = name;
    }
    PersonType.prototype.sayName = function() {
        console.log(this.name);
    }
    let person = new PersonType("Wahson");
    person.sayName();       // "Wahson"
    console.log(person instanceof PersonType);  // true
    console.log(person instanceof Object);  // true
    
    ```
- **ES6中的类**
    - 类声明，实际上类声明仅仅是一个语法糖。
    ```javascript
    class PersonClass {
        // 等价于PersonType的构造器
        constructor(name) {
            // 自有属性，该属性只出现在类的实例上，而不是原型上
            this.name = name;
        }
        // 等价于PersonType.prototype.sayName
        // 方法简写
        sayName() {
            console.log(this.name);
        }
    }
    
    let person = new PersonClass("Wahson");
    person.sayName();       // "Wahson"
    console.log(person instanceof PersonClass);  // true
    console.log(person instanceof Object);  // true
    
    ```
    - 类与自定义类型之间的区别
        - 类声明不会被提升，类声明的行为与let相似，程序执行到达声明处之前，类会存在于暂时性死区内。
        - 类声明中所有代码会自动运行在 _严格模式_  下，并且无法退出严格模式。
        - 类的所有方法都是不可枚举的
        - 类的所有方法内部区没有 ``[[Construct]]`` ，因此使用 ``new`` 来调用它们会抛出错误。
        - 调用类构造器时不使用 ``new`` 会抛出错误。
        - 试图在类的方法内部对类名重新赋值，会抛出错误。
- **类表达式**
    - 类表达式除了语法差异外，功能等价于类声明。
    ```javascript
    // 匿名类表达式
    let PersonClass = class {
        constructor(name) {
            this.name = name;
        }
        sayName() {
            console.log(this.name);
        }
    }
    
    ```
    - 具名类表达式
    ```javascript
    let PersonClass1 = class PersonClass2 {
        constructor(name) {
            this.name = name;
        }
        sayName() {
            console.log(this.name);
        }
    }
    console.log(typeof PersonClass1);   // "function"
    console.log(typeof PersonClass2);  // "undefined"
    ```
        PersonClass2标识符仅在类定义内部存在，因此只能用在类方法内部。
        
- **作为一级公民的类**
    - **在编程中，能被当做值来使用的就称为*一级公民*（first-class citizen）**。ES6中的类被定义为一级公民。
    ```javacript
    function createObj(classDef) {
        return new classDef();
    }
    
    let obj = createObj( class {
        sayHi() {
            console.log("Hi");
        }
    });
    obj.sayHi();   // "Hi"
    ```
- **定义访问器属性**
    - 用法
    ```javascript
    class CustomeHTMLElement {
        constructor(element) {
            this.element = element;        
        }
        get html() {
            return this.element.innerHTML;
        }
        set html(html) {
            this.element.innerHTML = html;
        }
    }
    
    ```
- **使用需计算的成员名**
    - 用法
    ```javascript
    let propName = "html";
    let methodName = "output"
    class CustomeHTMLElement {
        constructor(element) {
            this.element = element;        
        }
        get [propName]() {
            return this.element.innerHTML;
        }
        set [propName](html) {
            this.element.innerHTML = html;
        }
        
        [output]() {
            console.log(this.element);
        }
    }

    ```
- **生成器方法**
- **静态方法**
    - 静态方法：它的数据不依赖任何的实例。ES6中类的静成员创建只要在方法与访问器属性的名称前添加 ``static`` 标注。
    ```javascript
    class PersonClass {
        constructor(name) {
            this.name = name;
        }
        sayName() {
            console.log(this.name);
        }
        // 等价于PersonType.create
        static create(name) {
            return new PersonClass(name);
        }
        static get className() {
            return PersonClass.name;
        }
    }
    let person = PersonClass.create("Wahson");
    person.sayName();                       // "Wahson"
    console.log(PersonClass.className);    // "PersonClass"
    
    ```
- **类继承**
    - 用法
    ```javascript
    class Rectangle {
        constructor(length, width) {
            this.length = length;
            this.widht = width;
        }
        getArea() {
            return this.length * this.width;        
        }
        getGirth() {
            return (this.length + this.width) * 2;
        }
    }
    
    class Square extends Rectangle {
        constructor(width) {
            super(width, width);
        }
        // 派生类中的方法会屏蔽基类的同名方法
        getArea() {
            // 使用super调用基类中的方法
            return super.getArea();
        }
    }
    let square = new Square(2);
    console.log(square.getArea());   // 4
    console.log(square.getGirth());   // 8
    console.log(square instanceof Square);   // true
    console.log(square instanceof Rectangle);  // true
    
    ```
    - 继承了其他类的类称为派生类，派生类有几点需要注意的：
        - 如果派生类指定了构造器，就需要使用 ``super()``，否则会造成错误。
        - 如果派生类中没有构造器，``super()`` 方法会被自动调用，并会传入创建新实例时提供的所有参数作为参数。
        - ``super()`` 只能在派生类中使用。
        - 在构造器中，必须在访问this之前调用``super()``。由于super()负责初始化this，因此试图访问this会造成错误。
        - 唯一能避免使用super()的办法，是直接从类构造器中显式地返回一个对象。
        - 派生类也能继承静态方法，继承的方法与基类中的方法的行为相同。
- **从表达式中派生类**
    - 只有一个表达式能够返回一个具有[[Construct]]属性以及原型的函数，就可以对其使用extends。
    ```javascript
    // 定义一个ES5风格的构造器
    function Rectangle(length, width) {
        this.length = length;
        this.width = widht;
    }
    Rectangle.prototype.getArea = function() {
        return this.length * this.width;
    }
    
    function getBase() {
        return Rectangle;
    }
    // extends后面能接任意类型的表达式
    class Square extends getBase() {
        // ...
    }
    ```
    - 创建混入
    ```javascript
    let SerializableMixin = {
        serialize() {
            return JSON.stringify(this);
        }
    }
    let AreaMixin = {
        getArea() {
            return this.length * this.width;
        }
    }
    function mixin(...mixins) {
        let base = function(){}
        Object.assign(base.prototype, ...mixins);
        return base;
    }
    class Square extends mixin(AreaMixin, SerializableMixin) {
        constructor(width) {
            super();
            this.length = width;
            this.width = width;
        }
    }
    let square = new Square(3);
    console.log(square.getArea());     // 9
    console.log(square.serialize());   // "{length: 3, width: 3}" 
    ```
 - **继承内置对象**
    - ES6之前，并不能通过任何方式继承内置对象。但是在ES6中，这成为了可能。
    ```javascript
    // 继承了内置的Array
    class MyArray extends Array {
    }
    
    // 这里，MyArray的行为与Array完全相同
    let arr = new MyArray(1, 2, 3, 4);
    
    let subArr = arr.slice(1, 2);
    console.log(subArr instanceof MyArray);   // true
    ```
        继承内置对象有一个有趣的方面：任意能返回内置对象实例的方法，在派生类上能够自动返回派生类的实例。每当类实例的方法（构造器除外）必须创建一个实例时，类实例的构造器就会用为新实例的构造器。
- **Symbol.species**
    - 很多内置对象都定义了 ``Symbol.species`` 属性（如Array，ArrayBuffer，Map，PromiseRegExpSet），其返回值为 ``this``，意味着该属性总是返回自身的构造器函数。
    ```javascript
    // 内置类型使用Symbol.species 的方式类似于此
    class MyClass {
        static get [Symbol.species]() {
            return this;
        }
        constructor(value) {
            this.value = value;
        }
        clone() {
            return new this.constructor[Symbol.species](this.value);
        }
    }
    
    ```
        ``new this.constructor[Symbol.species]调用会返回MyClass``，clone()方法使用了该调用来返回一个新的实例，而没有直接使用MyClass，目的是允许派生类通过重写Symbol.species属性修改这个值。
        - 一般而言，每当想在类方法中使用this.constructor时，都应该设置类的Symbol.species属性，这么做能使派生类轻易地重写方法的返回值类型。
        - 如果一个拥有Symbol.species定义的类创建了派生类，要保证该类的方法返回类实例的时候使用此属性，而不是直接使用构造器。
- **new.target**
    - 在类构造器中通过 ``new.target`` 来判断类是如何被调用的。
    ```
    class Rectangle {
        constructor(length, width) {
            console.log(new.target === Rectangle);  
            this.length = length;
            this.widht = width;
        }
    }
    // new.target就是Rectangle
    let rec = new Rectangle(1,2); 
    
    ```