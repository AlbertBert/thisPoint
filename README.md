# thisPoint

javascript中的this对象是对于前端初学者来说是一个很让人头疼的东西。本文旨在帮助那些前端初学者能够能好的理解this，因此讲的并不是很深入，仅供大家参考。

借助阮一峰老师的话：它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。这句话最重要的是“运行时”三个字。如何理解？我们知道变量的作用域是在其声明的时候就已经确定了。也就是说一个变量在声明的时候我们就已经知道它的作用域链了，这就是所谓了编译时确定。相比之下，this对象则不是这样，它并不是在编写时就能确定，而是在this被调用时，也就是运行时才能被确定。因此在不同情况下，this所绑定的对象是不同的，所以this对象才会如此善变。

理解了this只有在运行时才能确定绑定的对象，我们再来看一下this绑定的几种情况。
* 默认绑定
* 隐式绑定
* 显式绑定
* new操作符绑定

上面四种情况的优先级递增的，后面我们会说明。我们先讲一下这四种基本情况。

### 1.默认绑定
这条规则你可以认为是当无法适用其他三条规则才会来使用的。一般常见的情况是，一般函数调用。注意这里的一般指的是这个函数并非作为对象的方法来调用。此时，this绑定到全局对象，对于浏览器来说就是window对象。但是要注意的是，如果是在严格模式下，this就不能绑定到window对象上了，而是会绑定到undefined。

比如下面这种常见的场景

    function printA() {
        console.log(this.a);
    }
    var a = 2;
    printA();      // 2
    
如果是在严格模式下，this会绑定到undefined

    function printA() {
        'use strict';
        
        console.log(this);      // undefined
        console.log(this.a);    //报错：Cannot read property 'a' of undefined
    }
    var a = 2;
    printA();
    
注意在严格模式下，this是否会绑定到全局对象是和函数的声明位置有关的，如果函数在声明的时候已经是严格模式了，那么this就会绑定到undefined，否则就是绑定到全局对象。

    function printA() {
        b = 3;
        console.log(this);      // Window
        console.log(this.a);    // 2
    }
    var a = 2;
     (function() {
        'use strict';
        console.log(this);      // undefined
        printA();
    })()
    
上面这种情况下，func是在全局作用域下声明的，并且全局作用域没有使用严格模式，所以this是绑定到全局对象的，不管func在哪里执行，func内部的this都会输出全局对象。

    var a = 2;
    (function() {
        'use strict';
        function printA() {
          console.log(this);      // Window
          console.log(this.a);    // 2
        }
        console.log(this);      // undefined
        printA();
    })()

而这种情况则是func在匿名函数的作用域下声明的，而且使用了严格模式，所以this绑定到了undefined

### 2.隐式绑定
当函数有调用的上下文对象时，this使用隐式绑定。通俗一点来说，就是当函数作为一个对象的方法来被调用时，this就被绑定到这个对象上。

    var obj = {
        a: 2,
        printA: function() {
          console.log(this);     // object: {a: 2, printA: ƒ}
          console.log(this.a);   // 2
        }
    }
    obj.printA();


如果对象的属性还是对象，多层调用的话，this绑定到最后一个或者说是最顶层的对象上

    var objA = {
        a: 2,
        objB: {
          a: 4,
          objC: {
            a: 8,
            printA: function() {
              console.log(this);     // object: {a: 2, printA: ƒ}
              console.log(this.a);   // 2
            }
          }
        }
    }
    objA.objB.objC.printA();
    
其实这种情况很好理解，因为最终其实是objC去调用printA的，所以this自然就绑定到了objC对象上了。

### 3.显式绑定
显式绑定就是强制将this绑定到自己想要绑定的对象上，而不是函数调用时的上下文对象。一般有三种方法可以实现显式绑定：call，apply和bind

call和apply两个方法基本相同，唯一的区别在于接收参数的方式不太一样，他们的第一个参数都是this要绑定的对象，但是后面的参数就有区别了。call的第二个到最后一个参数就是要传给执行函数的参数，而apply则是把这些传给函数的参数变成一个数组作为其第二个参数传入。也就是说call是执行函数的参数按顺序传递给call，而apply则是把参数放在数组里传递给apply

    var objA = {
        a: 2,
        add: function(n1, n2) {
          console.log(this.a + n1 + n2);
        }
    }
    var objB = {
        a: 4
    }
    objA.add(1, 3);                  // 6
    objA.add.call(objB, 1, 3);       // 8
    objA.add.apply(objB, [1, 3]);    // 8

上述例子表明通过call或apply可以强行改变this绑定的对象。本来objA调用add函数this是绑定到objA上的，通过call或apply可以将this绑定到objB。不同的是给call传参的时候一个一个传的，而apply则是作为一个数组传进去的。

bind和call、apply不同。当一个函数使用bind函数去强制绑定this时，会返回一个新函数，这个新函数功能和原来的一样，只不过里面的this已经被强制绑定了。而call和apply则是会直接之形函数。bind函数第一个参数也是要绑定的对象，其他的参数则是作为调用函数的参数。

    var objA = {
        a: 2,
        add: function(n1, n2) {
          console.log(this.a + n1 + n2);
        }
    }
    var objB = {
        a: 4
    }
    objA.add(1, 3);                         // 6
    objA.add.call(objB, 1, 3);              // 8
    objA.add.apply(objB, [1, 3]);           // 8
    var objBAdd = objA.add.bind(objB);      //objBAdd和objA.add功能一样
    objBAdd(1, 3);                          // 8
    var objBAdd2 = objA.add.bind(objB, 1);  //objBAdd2和objA.add功能一样,并且固定了第一个参数
    objBAdd2(3);                            // 8



bind和call、apply还有一个区别在于一旦用bind绑定了this，之后this是无法在绑定到别的对象了，也就是也就不能通过call和apply再去改变this绑定的对象了。而call和apply则没有这个限制。

    var objA = {
        a: 2,
        add: function(n1, n2) {
          console.log(this.a + n1 + n2);
        }
    }
    var objB = {
        a: 4
    }
    var objC = {
        a: 8
    }
    objA.add.call(objB, 1, 3);              // 8
    objA.add.apply(objB, [1, 3]);           // 8
    var objBAdd = objA.add.bind(objB);      //objBAdd和objA.add功能一样
    objBAdd(1, 3);                          // 8
    objBAdd.call(objC, 1, 3);               // 还是8，而不是变成8+1+3=12


自己实现的call和apply函数

    Function.prototype.myCall = function(obj) {
        obj.fn = this || window;
        var args = [];
        for (var i = 1;i< arguments.length;i++) {
          args.push(arguments[i]);
        }
        obj.fn(...args);
        delete obj.fn;
    }
    Function.prototype.myApply = function(obj) {
        obj.fn = this || window;
        obj.fn(...arguments[1]);
        delete obj.fn;
    }

### 4.new操作符绑定
在使用new操作服创建一个新的对象时，会有以下步骤
* 创建一个新对象
* 构造函数内部的this都绑定到这个新对象上
* 执行构造函数
* 返回这个新对象


    function Person(name) {
        this.name = name;
    }
    var wang = new Person('wang');
    console.log(wang.name);          // wang


现在说一下这四种情况的优先级
显然，默认绑定的优先级是最低的，在其他三种情况都不适用的情况才会去使用默认绑定。




