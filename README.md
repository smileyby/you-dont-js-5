作用域闭包
=========

接下来的内容需要对作用域工作原理相关内容和基础知识有非常深入的了解。

我们将注意力转移带这门语言中一个非常重要但又难以掌握，近乎神话的概念：闭包。如果你了解了之前关于词法作用域的讨论，那么闭包的概念几乎是不言而喻的。魔术师的幕布后藏着一个人，我们将要解开它的伪装。我们没说这个人是Crokford！

在继续学习之前，如果你还是对词法作用域相关内容有疑问，可以回顾下第2章中的相关内容，现在是一个好机会。

## 5.1 启示

对于那些有一点JavaScript使用经验但从未真正理解闭包概念的人来说，理解闭包可以看作是某种意义上的重生，但是需要付出非常多的努力和牺牲才能真正理解这个概念。

回忆我前几年的时光，大量使用JavaScript但却完全不理解闭包是什么。总是感觉这门语言有其隐蔽的一面，如果能够掌握将会功力大增，但讽刺的是我始终无法掌握其中的门道。还记得我曾经大量阅读早期框架的源码，试图能够理解闭包的工作原理。现在还能回忆起我的脑海中第一次浮现出关于“模块模式”相关概念时的激动心情。

那时我无法理解并且请进数年心血来探索，也就是我马上要传授你的秘诀：JavaScript中闭包无处不在，你只需要能够识别并拥抱它。闭包并不是一个需要学习的新的语法和模式才能使用的工具，它也不是一件必须接受Luke一样的原力训练才能使用和掌握的武器。

闭包是基于词法作用域书写代码时所产生的自然结果，你甚至不需要为了利用他们而有意地创建闭包。闭包的创建和使用在你的代码中随处可见。你缺少的是根据你的意愿来识别，拥抱和影响闭包的思维环境。

最后你恍然大悟：原来我的代码中已经到处都是闭包了。现在我终于理解他们了。节必报就好像Neo第一次见到矩阵一样。

## 5.2 实质问题

好了，夸张和浮夸的电影比喻已经够多了。

下面是直接了当的定义，你需要掌握它才能理解和识别闭包：

当函数可以记住并访问所在的此法作用域时，就产生了闭包，计时函数是在当前词法作用域之外执行。

下面用一些代码来解释这个定义。

```js

function foo() {
	var a = 2;
	function bar() {
		console.log( a ); // 2
	}
	
	bar();
}

foo();

```

这段代码看起来和嵌套作用域中的示例代码很相似。基于词法作用域的查找规则，函数bar()可以访问外部作用域中的变量a（这个例子中的是一个RHS引用查询）。

这是闭包吗？

技术上来讲，也许是。但根据前面的定义，确切的说并不是。我认为最准确地用来解释bar()对a的引用的方法是词法作用域的查找规则，而这些规则只是闭包的一部分。（但却是非常重要的一部分！）

从纯技术的角度说，在上面的代码片段中，函数bar()具有一个涵盖foo()作用域的闭包（事实上，涵盖了它能访问的所有作用域，比如全局作用域。）也可以认为bar()被封闭在foo()的作用域中。为什么呢？原因简单明了，因为bar()嵌套在foo()内部。

但是通过这种方式定义的闭包并不能直接进行观察，也无法明白在这个代码片段中闭包是如何工作的。我们可以很容易地理解此法作用于，而闭包则隐藏在代码之后的神秘阴影里，并不那么容易理解。

下面我们来看一段代码，清晰地展示了闭包：

```js

function foo() {
	var a = 2;
	function bar() {
		console.log( a );
	}
	
	return bar;
};
var baz = foo();
baz(); // a --- 朋友，这就是闭包的效果

```

函数bar()的词法作用域能够访问foo()的内部作用域。然后我们将bar()函数本身当做一个值类型进行传递。这个例子中，我们将bar所引用的函数对象本身当做返回值。

在foo()执行后，其返回值（也就是内部的bar()函数）赋值给变量baz并调用baz()，实际上只是通过不同的标识符引用调用了内部的函数bar()。

bar()显然可以被正常执行。但是这个例子中，它在自己定义的词法作用域意外的地方执行。

在foo()执行后，通常会期待foo()的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不再被使用的内存空间。由于看上去foo()内容不会被在使用，所以很自然会考虑对其进行回收。

而闭包的“神奇”之处正是可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回收。谁在使用这个内部作用域？原来是bar()本身在使用。

拜bar()所声明的位置所赐，它拥有涵盖foo()内部作用域的闭包，是的该作用域能够一直存活，一共bar()在之后任何时间进行引用。

bar()依然持有对该作用域的引用，而这个引用就叫做闭包。

因此，在几微秒之后变量baz被实际调用（调用内部函数bar），不出意料它可以访问定义时的词法作用域，因此它也可以入预期般访问变量a。

这个函数在定义是的词法作用域意外的地方被调用。闭包是的函数可以继续访问定义时的此法作用于。

当然，无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。

```js

function foo() {
	var a = 2;
	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // 妈妈快看呀， 这就是闭包！
}

```

把内部函数baz传递给bar，当调用这个内部函数时（现在叫做fn），它涵盖的foo()内部作用域的闭包就可以观察到，因为它能够访问a。

传递函数当然也可以是间接的。

```js

var fn;
function foo() {
	var a = 2;
	function baz() {
		console.log( a );
	}

	fn = baz; // 将baz分配给全局变量
}

function bar() {
	fn(); //妈妈快看呀， 这就是闭包！
}

foo();

bar(); // 2

```

无论通过何种手段将内部函数传递到所在作用域意外，它都会持有对原始定义作用域的引用，无论在何处这个函数都会形成闭包。

## 5.3 现在我懂了

前面的代码片段有点死板，并且为了解释如何使用闭包而人为地在结构上进行修饰。但我保证闭包绝不仅仅是一个好玩的玩具。你已经写过的代码中一定到处都是闭包的身影。

现在让我们来搞懂这个事实。

```js

function wait(message) {
	setTimeout( function timer() {
		console.log( message );
	}, 1000 )
}

wait( "Hello, closure!" );

```

将一个内部函数（名为timer）传递给setTimeout(...)。timer具有涵盖wait(...)作用域的闭包，因此还保有对变量message的引用。

wait(...)执行1000毫秒后，它的内部作用域并不会小时，timer函数依然保有wait(...)作用域的比闭包。

深入到引擎的内部原理，内置的工具函数setTimeout(...)持有对一个参数的引用，这个参数也叫作fn或者func，或者其他类似的名字。引擎会调用这个函数，在例子中就是内部的timer函数，而词法作用域在这个过程中保持完整。

这就是闭包。

或者，如果你很熟悉jQuery（或者其他能说明这个问题的JavaScript框架），可以思考下面的代码：

```js

function setupBot(name, selector) {
	$( selector ).click( function activator() {
		console.log( "Activating:" + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );

```

我不知道你会写什么样的代码，但是我写的代码负责控制由闭包机器人组成的整个全局无人机大军，这是完全可以实现的！

玩笑开完了，本质上无论何时何地，如果将函数（访问他们各自的此法作用域）当做第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、时间监听器、Ajax请求、跨窗口通信、Web Works或者任何其他的异步（或者同步）任务中，只要用了回调函数，实际上就是在使用闭包！

第三章介绍IIFE模式。通常认为IIFE是典型的闭包例子，但根据先前对闭包的定义，我并不是很统一这个观点。

```js

var a = 2;
(function IIFE() {
	console.log( a );
})();

```

虽然这段代码可以正常工作，但严格来讲它并不是闭包。为什么？因为函数（示例代码中的IIFE）并不是在它本身的此法作用域以外执行。它在定义时所在的作用域中执行（而外部作用域，也就是无安居作用域也持有a）。a是通过普通的此法作用于查找而非闭包被发现的。

尽管技术上来讲，闭包是发生在定义时的，但并不非常明显，就好像六祖慧能所说“既非风动， 亦非幡动， 仁者心动耳。”

尽管IIFE本身并不是观察闭包的恰当例子，但它的确创建了闭包，并且也是最常用来创建可以被封闭起来的比包工具。因此IIFE的确同闭包息息相关，计时本身并不会真的使用闭包。

亲爱的读者，现在把书放下，我有一个任务要给你，打开你最近写的JavaScript代码，找到其中的函数类型的值并指出哪里使用了闭包，即使你以前可能并不知道这就是闭包。

等你呦~

现在懂了吧！

## 5.4 循环和闭包

要说明闭包，for循环是最常见的例子。

```js

for (var i = 1; i <= 5; i += 1){
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 )
}

```

**由于很多开发者对闭包的概念认识得不是很清楚，因此当循环内部包含函数定义时，代码格式检查器经常发出警告。我们在这里介绍如何才能正确地使用闭包并发挥它的威力，但是代码格式检查器并没有那么灵敏，它会假设你并不是真正了解自己在做什么，所以无论如何都会发出警告。**

正常情况下，我们对这段代码行为的预期是分别输出数字1~5，每秒一次，每次一个。但实际上，这段代码在运行时会以每秒一次。的频率输出5次6.

这是为什么？

首先解释6是从哪里来的，这个循环的终止条件是i不在是<=5。条件首次成立时i的值是6。因此，输出显示的是循环结束时i的最终值。

仔细想一下，这好像又是显而易见，延迟函数的回调会在循环结束时才执行。事实上，当定时器运行时即使每个迭代中执行的setTimeout(..., 0)，所有的回调函数依然在循环结束才会被执行，因此会每次都输出一个6出来。

这里引申出一个更深入的问题，代码中到底有什么缺陷导致它的行为同语义所暗示的不一致呢？

缺陷是我们试图假设循环中的每一个迭代在运行时都会给自己“捕获”一个i的副本。但是根据作用域的工作原理，实际情况是尽管循环中的五个函数是在各个迭代中分别定义的，但是他们都被封闭在一个共享的全局作用域中，因此实际上只有一个i。

这样说的话，当然所有函数共享一个i的引用。循环结构让我们误以为背后还有更复杂的机制在起作用，但实际上没有。如果将延迟函数的回调重复定义五次，完全不适用循环，那它同这段代码是完全等价的。

下面回到正题。缺陷是什么？我们需要更多的闭包作用于，特别是在循环的过程中每个迭代都需要一个闭包作用于。

第三章介绍过，IIFE会通过声明立即执行一个函数来创建作用域。

我们来试一下：

```js

for (var i = 1; i <= 5; i += 1) {
	(function() {
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 )
	})();
}

```

这样能行吗？试试吧，我等着你。

我不卖关子了。这样不行。但是为什么呢？我们现在显然又有更多的此法作用于了。的确每个延迟函数都会将IIFE在每次迭代中创建的作用域封闭起来。

如果作用域是空的，那么仅仅将他们进行封闭是不够的。仔细看一下，我们的IIFE只是一个什么都没有的空作用于。它需要包含一点是指的内容才能为我们所用。

它需要有自己的变量，用来在每个迭代中存储i的值：

```js

for (var i = 1; i <= 5; i += 1){
	(function(){
		var j = i;
		setTimeout(function timer(){
			console.log( j );
		}, j*1000 );
	})();
}

```

行了！它能正常工作了！。

可以对这段代码进行一些改进：

```js

for (var i = 1; i <= 5; i += 1){
	(function(j){
		setTimeout(function timer(){
			console.log( j );
		}, j*1000);
	})(i);
}

```

当然，这些IIFE也不过就是函数，因此我们可以将i传递进去，如果愿意的话可以将变量名定为i，当然也可以还叫作i。无论如何这段代码现在可以工作了。

在迭代内使用IIFE会为每个迭代都生成一个新的作用域，是的延迟函数的回调可以将新作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。

问题解决啦~

### 重返块作用域

仔细思考我们队前面的解决方案的分析，我们使用IIFE在每次迭代时都会创建一个新的作用域。换句话说，每次迭代我们都需要一个块作用域。第三章介绍可let声明，可以用来劫持块作用域，并且这个块作用域中声明一个变量。

本质上这是将一个块作用域换成一个可以被关闭的作用域。因此，下面这些看起来很酷的代码就可以正常运行了：

```js

for (var i = 1; i <= 5; i += 1){
	let j = i;
	setTimeout(function timer(){
		console.log( j );
	}, j*1000);
}

```

但是，这还不是全部！for循环头部的let声明还会有一个特殊的行为。这个行为指出变量在循环中不止一次被声明，每次迭代都会声明。，随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

```js

for (let i = 1; i <= 5; i += 1){
	setTimeout(function timer(){
		console.log( i );
	}, i*1000);
}

```

很酷是吧？块作用域和闭包练手便是天下无敌。不知道你是什么情况，反正这个功能让我成为了一名快乐的JavaScript程序员。

## 5.5 模块

还有其他的代码模式使用闭包的强大威力，但从表面上看，他们似乎与回调无关。下面一起来研究其中最强大的一个：模块。

```js

function foo() {
	var someyhing = 'cool';
	var another = [1, 2, 3];
	
	function doSomething() {
		console.log( something )
	}

	function doAnother() {
		console.log( another.join( '!' ) );
	}
}

```

正如这段代码中所看到的，这里并没有明显的闭包，只是私有数据变量something和another，以及doSomething() 和 doAnother()两个内部函数，它们的词法作用域（而这就是闭包）也就是foo()的内部作用域。

接下来考虑一下代码：

```js

function CoolModule() {
	var something = 'cool';
	var another = [1, 2, 3];
	
	function doSomething() {
		console.log( something );
	}

	function doAnother(){
		console.log( another.join( ' ! ' ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother 
	};
}

var foo = CoolModule();
foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3

```

这个模式在JavaScript中被称为模块。最常见的实现模块模式的方法通常被称为模块暴露，这里展示的是其变体。

我们仔细研究一下这些代码。

首先，CoolModule() 只是一个函数，必须要通过调用它来创建一个模块实例。如果不执行外部函数，内部作用域和闭包都无法被创建。

其次，CoolModule() 返回一个用对象字面量语法`{ key: value, ... }`来表示对象。这个返回的对象中含有对内部函数而不是内部数据变量的引用。我们保持内部数据变量是隐藏且私有的状态。可以将这个对象类型的返回值看做本质上是模块的公用API。

这个对象类型的返回值最终被赋值给外部的变量foo，然后就可以通过它来访问API中的属性和方法，比如`foo.doSomething()`。

**从模块中返回一个实际的对象并不是必须的，也可以直接返回一个内部函数。jQuery就是一个很好的例子。jQuery和$标识符就是jQuery模块的公共API，但它们本身都是函数（由于函数也是对象，它们本身也可以拥有属性）。**

doSomething() 和 doAnother() 函数具有涵盖模块实例内部作用域的闭包（通过调用CoolModule()实现）。当通过返回一个含有属性引用的对象的方式来将函数传递到词法作用域外部时，我们已经创造了可以观察和实践闭包的条件。

如果要更简单的描述，模块模式需要具备两个必要的条件。

1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

一个具有函数属性的对象本身并不是真正的模块。从方便观察的角度看，一个从函数调用所返回的，只有数据属性而没有闭包函数的对象并不是真正的模块。

上一个实例代码中有一个叫做CoolModule()的独立的模块创建器，可以被调用任意多次，每次调用都会创建一个新的模块实例。当只需要一个实例时，可以对这个模式进行简单的改进来实现单例模式。

```js

var foo = (function CoolModule() {
	var something = 'cool';
	var another = [1, 2, 3];
	
	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( ' ! ' ) );
	}
	
	return {
		doSomething: doSomething,
		doAnother: doAnother
	}
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3 

```

我们将模块函数转换成IIFE，立即调用这个函数并将返回值直接赋值给单例的模块实例标识符foo。

模块也是普通的函数，因此可以接受参数：

```js

function CoolModule( id ) {
	function identify() {
		console.log( id );
	}
	
	return {
		identify: identify
	};
}

var foo1 = CoolModule('foo 1');
var foo2 = CoolModule('foo 2');
foo1.indetify(); // "foo 1"
foo2.identify(); // 'foo 2'

```

模块模式另一个简单但强大的变化用法是，命名将要作为公共API返回的对象：

```js

var foo = (function CoolModule( id ) {
	function change() {
		// 修改公共API
		publicAPI.identify = identify2;
	}
	
	function identify1() {
		console.log( id );
	}

	function indentify2() {
		console.log( id.toUpperCase() );
	}
	
	var publicAPI = {
		change: change,
		identify: identify1
	};
	
	return publicAPI;
})( 'foo module' );

foo.identify(); // foo module
foo.change(); 
foo.identify(); // FOO MODULE

```

通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对模块实例进行修改，包括添加或删除方法和属性，以及修改他们的值。

### 5.5.1 现代的模块机制

大多数模块依赖加载器/管理器本质上都是将这种模块定义封装进一个友好的API。这里并不会研究某个具体的库，为了宏观了解我会简单地介绍一些核心概念：

```js

var MyModules = (function Manager() {
	var modules = {};
	
	function define(name, deps, impl) {
		for (var i = 0; i < deps.length; i += 1){
			deps[i] = modules[deps[i]];
		}
	
		modules[name] = impl.apple( imp, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();

```

这段代码的核心是modules[name] = impl.apple(impl, deps)。为了模块的定义引入了包装函数（可以传入任何依赖），并且将返回值，也就是模块的API，存储在一个根据名字来管理模块列表中。

下面展示了如何使用它来定义模块：

```js

MyModules.define("bar", [], function(){
	function hello(who) {
		return "	Let me introduce:" + who
	}
	
	return {
		hello: hello
	};
});

MyModules.define("foo", ["bar"], function(bar) {
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() )
	}

	return {
		awesome: awsome
	};
});

var bar = MyModule.get("bar");
var foo = MyModules.get("foo");

console.log(
	bar.hello("hippo")
); // Let me introduce: hippo

foo.awsome(); // LET ME INTERDUCE: HIPPO

```

foo和bar模块都是通过一个返回公共API的函数来定义的。foo甚至接受bar的示例作为依赖参数，并能相应地使用它。

为了我们自己着想，应该多花一点时间来研究这些示例代码并完全理解闭包的作用吧。最重要的是理解模块管理器没有任何特殊的“魔力”。它们符合前面列出的模块模式的连个特点：为了函数定义引入包装函数，并保证它的返回值和模块的API保持一致。

换句话说，模块就是模块，计时在它们外层加上一个友好的包装工具也不会发生任何变化。

### 5.5.2 未来的模块机制

ES6中为模块增加了一级语法支持。但通过模块系统进行加载时，ES6会将文件单做独立的模块来处理。每个模块都可以导入其他模块或特定的API成员，同样也可以到处自己的API成员。

**基于函数的模块并不是一个能被稳定识别的模式（编译器无法识别），它们的API语义只有在运行时才会被考虑进来。因此可以运行时修改一个模块的API（参考前面关于公共API的讨论）。**

**相比之下，ES6模块API更加稳定（API不会在运行时改变）。由于编辑器知道这一点，因此可以在（的确也这样做了）编译期检查对导入模块的API成员的引用是否真实存在。如果API引用并不存在，编译期会在运行时跑出一个或多个“早期”错误，而不会像往常一样在运行期间采用动态的解决方案。**

ES6的模块没有“行内”格式，必须被定义在独立的文件中（一个文件一个模块）。浏览器或引擎有一个默认的“模块加载器”（可以被重载，但这远超出我们的讨论范围）可以在导入模块时异步地加载模块文件。

考虑一下代码：

bar.js

```js

function hello(who) {
	return "Let me introduce: " + who;
}

export hello

```

foo.js

```js

// 仅从bar模块导入hello()
import hello form "bar"
var hungry = "hippo";

function awsome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awsome;

```

baz.js

```js

// 导入完整的foo和bar模块

module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awsome(); // LET ME INTRODUCE: HIPPO

```

**需要用前面两个代码片段中的内容分别创建文件foo.js和bar.js。然后如第三个代码片段展示的那样，bar.js中的程序会加载或导入这两个模块并使用它们。**

import可以将一个模块中的一个或多个API导入到当前作用域中，分别绑定在一个变量上（在我们的例子里是hello）。module会将整个模块的API导入并绑定到一个变量上（在我们的例子里是foo和bar）。export会将当前模块的一个标识符（变量，函数）导出为公共API。这些操作可以在模块中根据需要使用人多次。

模块文件中的内容会被当作好像包含在作用域闭包中一样处理，就和前面介绍的函数闭包模块一样。

## 5.6 小结

闭包就好像从JavaScript中分离出来的一个充满神秘色彩的未开化世界，只有勇敢的人才能够到达那里。但实际上它只是一个标准，显然就是关于如何在函数作为值按需传递的词法环境中书写代码的。

当函数可以记住并访问所在的此法作用于，计时函数在当前词法作用域之外执行，这时就产生了闭包。

如果没能认出闭包，也不能了解它的工作原理，在使用它的过程中就很容易犯错，比如在循环中。但同时闭包也是一个非常强大的工具，可以用多种形式来实现模块等模式。

模块有两个主要的特征：（1）为了创建内部作用域而调用一个包装函数；（2）包装函数的返回值至少包括一对内部函数的引用，这样就会创建涵盖整个内部作用域的闭包。

现在我们会发现代码中到处都是闭包存在，并且我们能够识别闭包然后用他们来做一些拥有的事！















