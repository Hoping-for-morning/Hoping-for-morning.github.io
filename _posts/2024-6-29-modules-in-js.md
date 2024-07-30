---
title: Modules in Js 模块化
date: 2024-06-28
categories: [Front-end, modules]
tags: [frontend, modules, Js]     # TAG names should always be lowercase
# comments: false 
pin: true
math: true
mermaid: true
toc: true
---

### ToDoList
 
- CommonJs
- AMD
- CMD
- ESModule


### 模块化是什么？
把复杂系统分解到多个**离散的**功能**模块**

- 将一个复杂的程序依据一定的规则（**规范**）封装成几个块（文件），并组合在一起
- 块内部的数据相对而言是私有的，向外部暴露一些接口和外部其他模块通信

### 模块化的演进

#### 1. 全局模式

- **问题** ： 全局变量污染，命名空间冲突

```javascript
// module1.js
let data1 = 'module one data';
function foo() {
  console.log(`foo() ${data1}`);
}
function bar() {
  console.log(`bar() ${data1}`);
}

// module2.js
let data2 = 'module two data';
function foo() {
  //与模块1中的函数冲突了
  console.log(`foo() ${data2}`);
}

// 引入时，函数名冲突，后面的会覆盖前面的
<script type="text/javascript" src="module1.js"></script>
<script type="text/javascript" src="module2.js"></script>
```

#### 2. 单例模式（简单对象封装）

- 可以解决命名冲突的问题
- **问题** ：不安全，可以直接修改模块内的数据 `moduleOne.data`
  
```javascript
// module1.js
let moduleOne = {
  data: 'module one data',
  foo() {
    console.log(`foo() ${this.data}`);
  },
};

// module2.js
let moduleTwo = {
  data: 'module two data',
  foo() {
    console.log(`foo() ${this.data}`);
  },
};

<script type="text/javascript" src="module1.js"></script>
<script type="text/javascript" src="module2.js"></script>
  moduleOne.foo(); //foo() module one data
  moduleTwo.foo(); //foo() module two data
```

### IIFE （立即调用的函数表达式）

> 被定义后立即执行，通常用于创建一个独立的作用域，避免污染全局变量命名空间

- 可以解决模块暴露的问题，`module.data` 返回 undefined
- **问题** ：如果这个模块依赖另外一个模块怎么办

```javascript
(function(window) {
  // 数据
  let data = 'IIFE module data';

  //操作数据的函数
  function foo() {
    // 用于暴露的函数
    console.log(`foo() ${data}`);
  }

  function bar() {
    // 用于暴露的函数
    console.log(`bar() ${data}`);
    otherFun(); //内部调用
  }

  function otherFun() {
    // 内部私有的函数
    console.log('privateFunction go otherFun()');
  }

  // 暴露 foo 函数和 bar 函数
  window.module = { foo, bar };
})(window);
```

### IIFE 引入依赖

- **问题** ：必须严格按照顺序，依赖关系混乱，而且从外部看不出依赖关系

```javascript
// module1.js
(function(window, $) {
  //数据
  let data = 'IIFE Strong module data';

  //操作数据的函数
  function foo() {
    //用于暴露的函数
    console.log(`foo() ${data}`);
    $('body').css('background', 'red');
  }

  //暴露foo函数和bar函数
  window.moduleOne = { foo };
})(window, jQuery);
```

```html
<!-- test.html -->
<!--引入的js必须有一定顺序-->
<script type="text/javascript" src="jquery-1.10.1.js"></script>
<script type="text/javascript" src="module1.js"></script>
<script type="text/javascript">
  moduleOne.foo(); //foo() IIFE Strong module data  而且页面背景会变色
</script>
```

## 现在的一些模块化方案

### CommonJs
CommonJs 是服务器端模块的规范，node.js 采用了这个规范，用于浏览器时，需要用 `Browserify` 进行提前编译打包

- 引入方式使用 `require`
- 暴露方式使用 `module.exports` 和 `exports`
- 同步加载，有缓存

同步加载
: 使用 require 函数来加载一个模块时，Node.js 会暂停当前脚本的执行，直到所请求的模块被完全加载和编译

`module.exports` 和 `exports`的关系

: module 是一个全局变量，类似于在浏览器端的 Window 也是一个全局变量一样的道理。

```javascript
var exports = module.exports;
// exports 是对 module.exports 的引用，指向同一个对象
```

### ESModule
