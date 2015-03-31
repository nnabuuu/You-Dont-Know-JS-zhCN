# 你不懂JS：作用域和闭包

# 第五章：作用域闭包

在我们在讲述这一之前，希望你已经对作用域的作用机制有了清晰牢固的理解。

我们下面把注意力放到一个非常重要但难以捉摸的部分上：**闭包**。如果你一直跟随我们讨论了词法作用域，那么你所得到的好处就是理解起闭包来会非常简单。有一个男人在巫师的门帘之后，我们马上就会见到他。不，他的名字不是Crockford！（Douglas Crockford）

如果你对于词法作用域还有任何问题，在继续看下去之前，你应该趁这个机会回顾一下第二章。

## 启示

对于那些已经在JavaScript中有多年经验，但可能从未完全找过闭包的概念的人来说，了解闭包可能看起来就像是一种涅磐，必须通过努力和牺牲才能实现。

我回想起多年前我对于JavaScript有一个清楚的认知，但是完全不了解到底闭包是什么。有一种暗示一直在戏弄和嘲笑着我，它预示着这门语言还有另一面，了解它之后我能够获得比现在已知的更多的能力。我记得我通读了早期框架的源代码，尝试去理解它究竟是如何工作的。我记得第一次“模块化”的某些概念在我脑海中浮现时的场景。我清楚地记得那个“啊哈！”的瞬间。

当年我没有弄清楚的东西，花费了数年时间来理解的东西，现在我希望向你透露的秘密，是这个：**在JavaScript中，闭包无处不在，你只是需要认识它并拥抱它。**闭包不是一个你必须学习新的语法或模式才能够理解的东西。闭包甚至不是一种你必须掌握的武器，不像卢克掌控原力一样必须掌握。

闭包是编写代码而产生的，它依赖于词法作用域。它仅仅是产生。你不需要刻意地创建一个闭包来利用它的能力。闭包在你代码中无处不在地被创建并且被使用。你所缺失的，是适当的上下文来认识、拥抱并亲自使用闭包。

这个启示的时刻应该是：**噢，闭包已经在我代码中无处不在了，我终于可以看到它们了。**理解闭包就像尼奥第一次看见母体一样。

## 一些细节

好的，夸张而无耻的电影引用已经够了。

这里有一个你所需要的很接地气的定义来理解并认识闭包：

	闭包是一个函数可以在其词法作用域外执行时，仍然记住并访问它的词法作用域的能力。

让我们看看代码来说明这个定义。

	function foo() {
	    var a = 2;

	    function bar() {
	        console.log( a ); // 2
	    }

	    bar();
	}

	foo();

这段代码和我们之前讨论的嵌套作用域类似。函数`bar()`基于词法作用域的查询规则（这里是一个RHS查询）访问了外部闭包的变量`a`。

这是一个“闭包”吗？

呃，从技术层面上说... 可能是。但是从我们那个接地气的定义上看：并不完全是这样。我认为最准确的解释`bar()`引用`a`的方式是通过词法作用域查询规则来解释，这些规则只是闭包的一部分（请注意这里！）。

从纯学术的角度来看，上面这段代码所描述的是函数`bar()`拥有一个在`foo()`作用域之上的闭包（事实上，这个闭包也包括了它所访问到的作用域的其他部分，在我们这个例子中，也包括全局作用域）。略有不同的是，`bar()`在`foo()`的作用域之上。为什么？因为`bar`在`foo()`的内部出现。很简单吧。

但是，用这种方式定义的闭包并不是完全可见的，我们也没有在这段代码中看到闭包的使用。我们只是清楚的看到了词法作用域，但是闭包仍然处在代码神秘面纱的背后。

让我们用下面这段代码给闭包打一个全景光：

	function foo() {
	    var a = 2;

	    function bar() {
	        console.log( a );
	    }

	    return bar;
	}

	var baz = foo();

	baz(); // 2 -- Whoa, closure was just observed, man.
	
函数`bar()`可以访问`foo()`内部的作用域。但是然后，我们获取`bar()`，函数自身，并将它作为一个值传递。在这里我们`return`这个`bar`引用的函数对象自身。

在执行`foo()`之后，我们将它的返回值（内部的`bar`函数）赋值给名为`baz`的便利，然后我们调用`bar()`的时候，实际上是调用了我们的内部函数`bar()`，只是通过另一个不同的标识符而已。

`bar()`被执行了，当然。但是在这个例子中，它是在它所定义的词法作用域的外部被执行的。

在`foo()`执行之后，我们普遍会认为整个`foo()`的内部作用域将会消失，因为我们知道引擎有一个垃圾回收器来回收不需要了的内存。因为`foo()`的内容不再被使用了，因此它们看起来很自然地应该被回收掉。

但是闭包的“神奇”之处在于，它阻止了这一切的发生。内部的作用域实际上仍然在“被使用”，因此不会被回收。谁在使用它？**函数`bar()`自身。**

拜其声明地点所赐，`bar()`在`foo()`的内部作用域上拥有一个词法作用域的闭包，它为`bar()`保存着这个作用域以便之后需要使用。

**`bar()`仍然拥有一个对该作用域的引用，这个引用被称为闭包。**

因此，几微秒之后，当变量`baz`被调用（实际是我们在内部声明的`bar`函数被调用）时，`bar()`就能够访问它在编写时所带有的词法作用域，因此它能够像我们期望的一样访问变量`a`。

函数在其自身定义的词法作用域外可以被调用，这归功于**闭包**允许函数可以继续访问编写时所定义的词法作用域。

当然，有很多种函数作为值被传递，然后再另一个地方调用的例子。它们都可以用闭包来解释。

	function foo() {
	    var a = 2;

	    function baz() {
	        console.log( a ); // 2
	    }

	    bar( baz );
	}

	function bar(fn) {
	    fn(); // look ma, I saw closure!
	}
	
我们将内部函数`baz`传递给`bar`，并且调用该内部函数（现在名为`fn`），当我们这样做时，我们可以看到它在在`foo()`作用域内部的闭包，因为它可以访问到`a`。

这些传递的函数也可以是非直接的。

	var fn;

	function foo() {
	    var a = 2;

	    function baz() {
	        console.log( a );
	    }

	    fn = baz; // assign `baz` to global variable
	}

	function bar() {
	    fn(); // look ma, I saw closure!
	}

	foo();

	bar(); // 2
	
无论我们采用什么方式将内部的函数传递到它的词法作用域之外，它都会仍然保留它对它原始声明处的作用域的引用，并且无论我们在什么地方执行它，闭包都会被使用到。

## 现在我能看见了

前面的代码有一些学术化了，它们是特地写来让你理解闭包的。但是我向你保证，闭包不仅仅只是一个新的玩具。闭包是在你的代码中无处不在的。让我们看看事实真相。

	function wait(message) {

	    setTimeout( function timer(){
	        console.log( message );
	    }, 1000 );

	}

	wait( "Hello, closure!" );
	
我们使用了一个内部函数（名为`timer`）并将其传递给了`setTimeout(..)`。但是`timer`在`wait(..)`的作用域之上有一个作用域闭包，它不停的保持并使用对变量`message`的引用。

在我们执行`wait(..)`的一千毫秒之后，它的内部作用域不会消失，因为匿名函数仍然在该作用域上持有一个闭包。

我们深入到引擎的内部看看，内置的功能类`setTimeout(..)`有对一些参数的引用，可能叫做`fn`也可能叫做`func`。引擎会调用这个函数，然后它实际上是调用我们内部的`timer`函数，并且词法作用域的引用依然是完好无损的。

**闭包。**

或者，如果你信仰jQuery（或者任何JS框架，无所谓）

	function setupBot(name,selector) {
	    $( selector ).click( function activator(){
	        console.log( "Activating: " + name );
	    } );
	}

	setupBot( "Closure Bot 1", "#bot_1" );
	setupBot( "Closure Bot 2", "#bot_2" );
	
我不知道你会写那种代码，但是我通常会编写那种负责管理整个全局机器人军队的闭包机器人，这是完全可行的！

说笑归说笑，本质上来说，只要你将函数（它们访问自己的词法作用域）视为第一类元素并将其传递出去，你往往都会利用到闭包。它可以是计时器、事件句柄、Ajax请求、跨窗口消息、Web worker、或任何异步（或者同步！）的任务，当你传递一个回调函数时，准备好带上一些闭包吧！

**注：**第三章我们介绍了IIFE模式。尽管很多人认为IIFE（自身）是一个可见闭包的例子，从我们前面的定义来看，我并不完全同意。

	var a = 2;
	
	(function IIFE(){
	    console.log( a );
	})();

这段代码可以工作，但是它严格来说并不是一个可见的闭包。为什么？因为函数（这里我们将其命名为IIFE）并不在它的词法作用域外被调用。它仍然在其声明的原地被调用（外部的全局作用域仍然持有`a`）。`a`是通过词法作用域的查找被发现的，而不是真正的通过闭包发现。

尽管从技术层面上说闭包可能发生在声明时期，但严格来说并不是这样，就像有人说的那样，这就像森林中有一棵树倒下而没有人听到一样。

尽管IIFE不是一个闭包的例子，它实际上创建了作用域，并且是一种非常普遍的用来创建可关闭的作用域的工具。因此IIFE是和闭包紧密相关的，虽然它自身并不使用闭包。

看到这里请把书放下，亲爱的读者，现在我有一个任务交给你。打开你最近一次编写的JavaScript代码。寻找将函数作为参数值传递的地方，分辨那些你以前不知道而实际上确实用到了的闭包。

我会在这里等你。

现在，你看见了！

## 循环 + 闭包

最典型的说明闭包的例子来自于for循环。

	for (var i=1; i<=5; i++) {
	    setTimeout( function timer(){
	        console.log( i );
	    }, i*1000 );
	}

**注：**当你把函数放在for循环中时，代码检查工具总是会抱怨，因为对于闭包的不理解是**在程序员中是非常普遍的。**我们会在这里解释如何恰当的实现它，利用到闭包的全部能力。但是无论怎样代码检查工具还是会抱怨，如有你不真正知道你在做什么的话。

这段代码本意是要每隔一秒打印一个数字：“1”、“2”、...“5”。

事实上，如果你执行这段代码，你会打印"6"5次，每次间隔一秒。

哈？

首先，让我们解释一下这里的`6`是从何而来的。for循环的结束条件是当`i`并不是`<=5`。因此这就是`i`等于6的原因。实际上这段代码输出的是循环结束之后的`i`。

再看一遍，这实际上非常清楚。timeout函数回调是在整个循环执行后才运行的。实际上，即使你使用的是`setTimeout(..., 0)`，这些回调仍然会在循环之后执行，并且每个都会打印`6`。

这里有一个更深入的问题了：你的代码中遗漏了什么才会使得结果变成这样呢？

我们遗漏的是我们以为每个循环周期都会“捕获”它自己的`i`的拷贝。但是，由于作用域的原因，虽然这5个函数都分别在不同的迭代周期中定义，**所有的闭包都持有同一个全局作用域**，也就是说，只有唯一一个`i`。

因此，所有的函数都分享同一个`i`的引用。循环的结构让我们以为会发生一些更为复杂的机制。而实际上并没有。这样写和根本不用循环而将每个回调一个个的声明没有区别。

好，现在回到我们的问题。我们遗漏了什么？我们需要更多的闭包作用域。具体来说，我们需要为每一次循环迭代创建一个新的作用域闭包。

我们在第三章了解到IIFE可以通过声明并立即执行一个函数的方式创建一个闭包。

让我们试试这样做。

	for (var i=1; i<=5; i++) {
	    (function(){
	        setTimeout( function timer(){
	            console.log( i );
	        }, i*1000 );
	    })();
	}

这样有效吗？再试试吧，我会等你的。

让我来打消你的疑虑。**这并不奏效。**为什么？我们现在已经有了更多的词法作用域，每一次timeout的回调函数执行时都使用了通过IIFE创建的自己的作用域。

事实上，**如果作用域为空时**，只创建一个作用域是没有任何用处的。仔细看，我们的IIFE只是创建了一个空的作用域。我们需要在里面放置一些东西来帮助我们。

它所需要的是自己的变量，每个迭代中都需要一个`i`的拷贝。

	for (var i=1; i<=5; i++) {
	    (function(){
	        var j = i;
	        setTimeout( function timer(){
	            console.log( j );
	        }, j*1000 );
	    })();
	}

**我发现了！这样是有效的！**

更多人喜欢的是这样的微调后的版本：

	for (var i=1; i<=5; i++) {
	    (function(j){
	        setTimeout( function timer(){
	            console.log( j );
	        }, j*1000 );
	    })( i );
	}

当然，因为这些IIFE只是函数，所以我们可以将`i`传入，而将其命名为我们喜欢的`j`，我们甚至也可以同样将其称为`i`，现在这段代码可以工作了。

每个循环中使用的IIFE都该迭代周期创建了一个新的作用域，它使得我们的timeout函数的回调有能力在每个循环中都访问一个新的作用域，每个作用域中都有一个正确的迭代值可以访问。

问题解决！

## 重新审视块作用域

认真看一下我们前面这个解决方案的分析过程。我们使用了一个IIFE来为每一次迭代创建一个新的作用域。换句话说，我们实际上需要一个每个迭代周期的**块作用域**。第三章我们演示了`let`声明，它劫持了一个块，并在块中声明了一个变量。

**其本质是将一个块转化为一个作用域以便我们可以覆盖。**因此，下面这段神奇的代码是“可以工作的”:

	for (var i=1; i<=5; i++) {
	    let j = i; // yay, block-scope for closure!
	    setTimeout( function timer(){
	        console.log( j );
	    }, j*1000 );
	}

但是，不仅仅是这样！（用我最Bob Barker的声音说）。对于`let`声明处在for循环头的场景有一个特殊的定义。该行为表示变量声明不仅仅会为循环声明一次，并且会在**每次循环迭代中都声明一次。**因此，它会像我们期望那样在每个自迭代中都用新值初始化一次。

	for (let i=1; i<=5; i++) {
	    setTimeout( function timer(){
	        console.log( i );
	    }, i*1000 );
	}

很酷不是吗？将块作用域和闭包相结合，我们能解决所有的问题。我不知道你怎么想，但是这让我变成了一个快乐的JavaScript程序员。

## 模块

有一些其它的代码模式可以发挥闭包的作用，而表面上并没有用到回调。让我们看看其中一个很强大的例子：模块。

	function foo() {
	    var something = "cool";
	    var another = [1, 2, 3];
	
	    function doSomething() {
	        console.log( something );
	    }
	
	    function doAnother() {
	        console.log( another.join( " ! " ) );
	    }
	}

在这个例子中，我们没有发现可见的闭包。我们只有一些私有数据变量`something`和`another`，以及一些内部函数`doSomething()`和`doAnother()`，它们都在`foo()`的作用域内有词法作用域（以及闭包）。

但是下面让我们看看这样：

	function CoolModule() {
	    var something = "cool";
	    var another = [1, 2, 3];
	
	    function doSomething() {
	        console.log( something );
	    }
	
	    function doAnother() {
	        console.log( another.join( " ! " ) );
	    }
	
	    return {
	        doSomething: doSomething,
	        doAnother: doAnother
	    };
	}
	
	var foo = CoolModule();
	
	foo.doSomething(); // cool
	foo.doAnother(); // 1 ! 2 ! 3

JavaScript中这种模式我们成为模块。最普遍的实现这种模块模式往往被称为“Revealing Module”，也就是我们在这里呈现的这种变种。

让我们来查看一下这段代码。

首先，`CoolModule()`只是一个函数，但是它需要被调用以便产生一个模块实例。如果没有外部函数的执行，内部的作用域和闭包就不会被创建。

然后，`CoolModule()`函数返回了一个对象，由文本对象语法`{ key: value, ...}`来定义。我们返回的对象有一个对我们内部函数的引用，但是没有对内部数变量的引用。我们将其视为隐藏私有。将这个返回对象视为**我们模块的公有API**是非常恰当的。

这个对象最终被赋值给外部变量`foo`，然后我们可以访问这些API上的属性方法，就像`foo.doSomething()`一样。

**注：**从模块中返回一个实际对象并不是所必须的。我们可以仅仅直接返回一个内部函数。jQuery实际上就是一个好的例子。`jQuery`和`$`标识符是jQuery模块的公有API，但是它们自身实际上只是函数（它们自身可以有属性，因为所有的函数实际上都是对象）。

`doSomething()`和`doAnother`函数拥有在模块“实例”（通过调用`CoolModule()`得到）的内部作用域之上的闭包。当我们将这些函数通过返回对象的属性的方式传递到词法作用域之外时，我们并不知道哪一个闭包会被使用：

1. 一定有一个外部包装函数，它会被至少调用一次（每次创建一个新的模块实例）

2. 包装函数需要至少返回一个内部函数，因此这个内部函数就拥有了私有作用域之上的闭包，并且可以访问或修改私有状态。

一个仅仅包含函数属性的对象并不是一个模块，一个从函数中返回的仅仅带有数据属性而没有闭包函数的对象也不是一个模块。

上面的代码显示了一个单独的名为`CoolModule()`的模块构造器，它可以被调用任意多次，每次都会创造一个新的模块实例。一个小变种是当你只想要一个实例，也就是所谓的“单例模式”：

	var foo = (function CoolModule() {
	    var something = "cool";
	    var another = [1, 2, 3];
	
	    function doSomething() {
	        console.log( something );
	    }
	
	    function doAnother() {
	        console.log( another.join( " ! " ) );
	    }
	
	    return {
	        doSomething: doSomething,
	        doAnother: doAnother
	    };
	})();
	
	foo.doSomething(); // cool
	foo.doAnother(); // 1 ! 2 ! 3

这里，我们将函数变为一个IIFE（见第三章），然后我们立即调用它并将其返回值赋值给实例标识符`foo`。

模块都只是函数，因此它们可以接收参数：

	function CoolModule(id) {
	    function identify() {
	        console.log( id );
	    }
	
	    return {
	        identify: identify
	    };
	}
	
	var foo1 = CoolModule( "foo 1" );
	var foo2 = CoolModule( "foo 2" );
	
	foo1.identify(); // "foo 1"
	foo2.identify(); // "foo 2"

另一个细微但强大的模块模式的变种是将你所要返回的对象命名为你的公共API：

	var foo = (function CoolModule(id) {
	    function change() {
	        // modifying the public API
	        publicAPI.identify = identify2;
	    }
	
	    function identify1() {
	        console.log( id );
	    }
	
	    function identify2() {
	        console.log( id.toUpperCase() );
	    }
	
	    var publicAPI = {
	        change: change,
	        identify: identify1
	    };
	
	    return publicAPI;
	})( "foo module" );
	
	foo.identify(); // foo module
	foo.change();
	foo.identify(); // FOO MODULE

通过保存对模块实例内部公有API对象的引用，你可以通过添加/y移除方法、属性或更改属性值的方式从内部修改该模块实例。

## 现代模块

各种各样的模块加载器/管理器实质上都是包装了这一模式使其拥有更友好的API。不专注于某个特定的库，让我来给你演示一个非常简单的内容来证明（**只是证明**）。

	var MyModules = (function Manager() {
	    var modules = {};
	
	    function define(name, deps, impl) {
	        for (var i=0; i<deps.length; i++) {
	            deps[i] = modules[deps[i]];
	        }
	        modules[name] = impl.apply( impl, deps );
	    }
	
	    function get(name) {
	        return modules[name];
	    }
	
	    return {
	        define: define,
	        get: get
	    };
	})();

这段代码的核心部分是`modules[name] = impl.apply(impl, deps)`。它是在调用模块的包装定义函数（通过传递依赖的方式），并将返回值，即模块的API，存储到一个由名称跟踪的模块内部的列表上。

然后下面是我可能会如何使用它来定义一些模块：

	MyModules.define( "bar", [], function(){
	    function hello(who) {
	        return "Let me introduce: " + who;
	    }
	
	    return {
	        hello: hello
	    };
	} );
	
	MyModules.define( "foo", ["bar"], function(bar){
	    var hungry = "hippo";
	
	    function awesome() {
	        console.log( bar.hello( hungry ).toUpperCase() );
	    }
	
	    return {
	        awesome: awesome
	    };
	} );
	
	var bar = MyModules.get( "bar" );
	var foo = MyModules.get( "foo" );
	
	console.log(
	    bar.hello( "hippo" )
	); // Let me introduce: hippo
	
	foo.awesome(); // LET ME INTRODUCE: HIPPO

"foo"和"bar"模块都由一个返回公共API的函数来定义。"foo"甚至接受一个"bar"的实例作为依赖参数，并且可以使用它。

你应该多花点时间查看上面的代码片段以便你可以完全了解闭包的强大之处，并按你所需的来使用它们。你要记住的重点是并模块管理器并没有任何特别的“魔法”。它们同时满足满足我上面列出的模块模式的特征：调用一个函数定义包装器，持有它的返回值作为模块的API。

换句话说，模块就只是模块，即使你将一个友好的包装工具放在它们之上。

## 未来模块

ES6位模块的概念加入了第一类的语法支持。当通过模块系统加载时，ES6将一个文件视为一个单独的模块。每一个模块可以导入其他的模块或者一个具体的API成员，也可以暴露自己的公有API成员。

**注：**基于函数的模块并不是静态可识别的模式（即编译器可认知的模式），因此它们的API语义知道运行时才会被认识。因此，你可以实际上在运行时修改一个模块的API（见稍早时候对`publicAPI`的讨论）。

相比之下，ES6模块的API是静态的（API并不在运行时改变）。因为编译器知道这一点，它可以（并且的确这样做了！）在编译（读取文件）时期进行检验，查看对于导入的模块的引用是否实际存在。如果API引用不存在，编译器会在编译器就抛出一个早期的错误，而不是等到传统的动态运行时才确定。

ES6模块**没有**“内联”格式，它们必须在单独的文件中定义（每个模块需要一个文件）。浏览器/引擎有一个默认的“模块加载器”（这是可重写的，但是这和我们这里的讨论没有太大关系了），当它被载入时，它会同步地一个个加载模块文件。

考虑下面的代码：

**bar.js**

	function hello(who) {
	    return "Let me introduce: " + who;
	}
	
	export hello;

**foo.js**

```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
    console.log(
        hello( hungry ).toUpperCase()
    );
}

export awesome;
```


```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
    bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**注：**你必须创建两个单独的文件**"foo.js"**和**"bar.js"**，它们的内容分别如上所示。然后你的程序就可以读取/导入这些模块来使用它们了，就像第三块代码那样。

`import`可以将模块的一个或多个API导入到当前作用域中，每个API对应一个绑定的变量（在我们这个例子中是`hello`）。而`module`会导入整个API到一个绑定变量中（在我们的例子中是`foo`、`bar`）。`export`导出一个标识符（变量或函数）到当前模块的公有API中。这些操作可以在一个模块定义中被多次使用。

模块文件中的内容被视为封闭在作用域闭包中，就像我们之前看到的函数闭包模块那样。

# 章节回顾 （给太长不看的人看的）

闭包似乎是JavaScript中一个单独神秘的不为人知的世界，只有少数勇敢的人才能触及。但实际上，它只是一个种标准，是一种我们在词法作用域环境中编写代码时的显而易见的事实，这里函数和变量可以根据需求随意传递。

**闭包是一个函数即使在它在词法作用域之外被调用时仍然能记住并访问自己的词法作用域。**

如果我们不小心辨识并理解其工作原理的话，闭包可能会把我们弄晕，例如在循环中使用它们的时候。但是它们同时也是无比强大的工具，它们以各种各样的形式构成了多种模式，例如模块模式。

模块要求它有两个关键特性：1. 需要调用一个外部包装函数来创建外部作用域。 2. 包装函数的返回值必须包含至少一个内部函数的引用，这样这个内部函数就具有在包装函数的私有内部作用域之上的闭包。

现在我们可以在已有代码中看出闭包了，我们也拥有了识别和利用它们的能力！