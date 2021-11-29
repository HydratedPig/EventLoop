---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
title: Event Loop
---

# [Event Loop](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    <uim-rocket class="text-xl text-white-900 animate-pulse" />
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <!-- <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button> -->
  <a href="https://github.com/HydratedPig/EventLoop" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<style>
.slidev-layout h1  {
  -webkit-text-fill-color: unset;
}
h1 a:hover{
  color: white !important;
}
</style>

---

# 单线程的JavaScript
JavaScript从诞生之日起就是一门单线程的非阻塞的脚本语言.
<br/>

<div v-click="1">
为什么是单线程?
</div>

<br/>

<div v-click="2">
试想一下多线程，那么当两个线程同时对dom进行一项操作，例如一个向其添加事件，而另一个删除了这个dom，此时该如何处理呢？因此，为了保证不会 发生类似于这个例子中的情景，JavaScript选择只用一个主线程来执行代码，这样就保证了程序执行的一致性。<br/>
</div>

<br/>

<v-click at="3">
```mermaid {theme: 'neutral', scale: 0.8}
flowchart LR
    开始线程 --> 任务一 --> 任务二 --> 任务三 --> 任务四 --> 结束线程
```
</v-click>

<v-click at="4">

```go
package main
import "fmt"
func main() {
	task1 := 1
	task2 := task1 + 1
	task3 := task2 + 1
	fmt.Println("task4", task3, task2, task1)
}
```

</v-click>

<arrow v-click="5" x1="200" y1="360" x2="320" y2="266" color="#564" width="3" arrowSize="1" />
<div v-click="5" class="fixed top-80 left-60"><span class="text-3xl">?</span>如何插入一个任务</div>

---

# 事件循环(Event Loop)
```go
package main
import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	reader := bufio.NewReader(os.Stdin)
	for {
		fmt.Println("请输入文本")
		text, _ := reader.ReadString('\n')
		fmt.Println("文本结果:", text)
	}
}
```

<v-click at="1">

这种设计模型有哪些优点呢?

</v-click>
<v-click at="2">

- 引入**循环机制**,通过 while 循环,代码可以一直执行下去
- 引入**事件**,可以接受用户的输入事件并输出信息<br/>
这样子线程就可以"永动"了

</v-click>

---

# 消息队列

刚才的示例中,引入了事件循环机制,线程在执行过程可以接收新的任务执行了,但是,任务都来自于线程内部,如果想接收其他线程发送过来的任务,这种模型是无法做到的<br/>
除了引进事件循环,我们还需要引入消息队列,这样才能让浏览器动起来
<br/>

<div class="text-center w-full justify-center flex flex-wrap">
  <div><img v-click="1" src="/assets/0.jpg" style="height:280px;" class="object-cover"/></div>
  <p v-click="1" class="w-full text-sm">《浏览器工作原理》 15-消息队列和事件循环</p>
</div>

---

# 处理 DOM 事件
试想一下一个典型的场景,监听 DOM 树的变化,并处理相关的业务逻辑.

<v-click at="1">

```js
function listener() {
  console.log('body中子元素被修改');
}
document.body.addEventListener(
  'DOMSubtreeModified',
  listener
);
function appendBody() {
  appendBody.count++;
  document.body.append(`text${appendBody.count}\n`);
}
appendBody.count = 0;
function doOtherTasks() {
  console.log('other tasks');
}
appendBody();
appendBody();
appendBody();
doOtherTasks();
```

</v-click>

---

# 如何权衡处理高优先级任务?

显而易见，刚才例子中 listener 的业务逻辑会阻塞后续的任务执行。如果 listener 业务逻辑处理量大并且和后续任务关联性不大，那么我们没有必要让这些任务阻碍后续业务逻辑的执行，导致执行效率下降。但是如果我们将异步的消息添加到消息队列尾部，又会有新的问题，监控的时效性丢失。那么如何兼顾当前任务的时效性和监控的实时性呢？

<v-click at="1">

### 没错，微任务应运而生

</v-click>

<v-click at="2">

我们把刚才的代码稍作更改

</v-click>

---

```js
function asyncListener() {
  queueMicrotask(() => {
    console.log('body中子元素被修改');
  })
}
document.body.addEventListener(
  'DOMSubtreeModified',
  asyncListener
);
function appendBody() {
  appendBody.count++;
  document.body.append(`text${appendBody.count}\n`);
}
appendBody.count = 0;
function doOtherTasks() {
  console.log('other tasks');
}
appendBody();
appendBody();
appendBody();
doOtherTasks();
```
<v-click at="1">

这样我们在宏任务执行过程中，将 DOM 变化的监听丢进微任务里，在当前宏任务结束之前执行微任务检查去执行微任务，这样既不会影响宏任务继续执行，又保障了监听的时效性，可谓是一举两得。

</v-click>

---

# 宏任务

- 渲染事件(如解析DOM、计算布局、绘制); 
- 用戶交互事件(如鼠标点击、滚动⻚面、放大缩小等);
- JavaScript脚本执行事件;
- 网络请求完成、文件读写完成事件;
- 定时任务（setTimeout、setInterval）。

<v-click at="1">

为了协调这些任务有条不紊地在主线程上执行，⻚面进程引入了消息队列和事件循环机制，渲染进程内部会维护多个消息队列，比如延迟执行队列和普通的消息队列。然后主线程采用一个for循环，不断地从这些任务队列中取出任务并执行任务。我们把这些消息队列中的任务称为宏任务。

</v-click>

---

# [WHATWG 规范定义](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
宏任务执行过程
- 从 taskQueue 中取出第一个 runnable task 称之为 oldestTask，并将它从 taskQueue 中移除
- 将 oldestTask 设置为 Event Loop 当前正在执行的任务，并记录 taskStartTime
- 执行 oldestTask
- 执行微任务检查点
- 任务完成后统计执行完成的时⻓等信息。

<v-click at="1">

我们刚才提过，作为宏任务放到消息队列中，无法保障时效性，除了无法保障时效性，我们也很难控制任务开始的时间。

</v-click>

---

```js
setTimeout(() => {
  const id = setInterval(() => console.log('interval'), 0)
  function timerCallback4(){
    console.log(4);
  }
  function timerCallback3(){
    console.log(3);
    setTimeout(timerCallback4,0);
  }
  function timerCallback2(){
    console.log(2);
    setTimeout(timerCallback3,0);
  }
  function timerCallback(){
    console.log(1);
    setTimeout(timerCallback2,0);
  }
  setTimeout(timerCallback,0);

  setTimeout(() => clearInterval(id),100);
},4000)
```

<v-click at="1">

很明显 setTimeout 特别容易被插队

</v-click>

<v-click at="2">

试想一下，如果中间被插入的任务执行时间过久的话，那么就会影响到后面任务的执行了。

</v-click>

---

# 微任务

在刚才提到的 [WHATWG](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint) 规范中，我们可以知道**微任务就是一个需要异步执行的函数，执行时机是在主函数执行结束之后、当前宏任务结束之前。**

<v-click at="1">

微任务检查点的执行
- 将 perform a microtask checkpoint 设置为 true
- 如果 microtask queue 非空
  - 微任务队列出队 oldestMicrotask 
  - 当前事件循环执行的任务中执行 oldestMicrotask
  - 再次执行微任务检查点
  - 将事件循环当前运行的任务设置为空。
- 将 perform a microtask checkpoint 设置为 false

</v-click>

<v-click at="2">

现代浏览器中主要有 MutationObserver， Promise，queueMicrotask 产生微任务

</v-click>

---

# 回调地狱
```js
function xhrFetch(data, resolve, reject) {
  // ...
}
xhrFetch({},
  function resolve() {
    xhrFetch({},
      function resolve2() {
        xhrFetch({},
          function resolve3() {
            // ...
          },
          function reject3() {
            // ...
          })
      },
      function reject2() {
        // ...
      })
  },
  function reject1() {
    // ...
  })
```

---

# Promise

```js
function xhrFetch(data) {
  // ...
}
const x0 =  xhrFetch({});
const res1 = x0.then(() => {
  return xhrFetch({});
})
const res2 = res1.then(() => {
  return xhrFetch({});
})

res2.catch(error => {
  console.log(error);
})
```
<v-click at="1">

我们可以看到，引入 Promise 后代码变得线性，错误处理也可以被合并到一起

</v-click>

---

# Promise 与微任务

```js
console.log(0);
new Promise(resolve => {
  console.log(1);
  resolve(3);
  console.log(2);
}).then(res => {
  console.log(res);
})
```
<v-click at="1">

众所周知，在 Promise/A+ 规范中：<br/>
- then是在 promise 状态转为 fulfilled 或者 rejected 才会被调用
- then 有两个参数 onFulfilled 和 onFulfilled
- onFulfilled 或 onRejected 不能在执行上下文栈有任务时被调用

</v-click>

<v-click at="2">

根据第三点，易知，```Promise.resolve().then()```需要在下一个任务执行的时机执行，那么这个任务既可以是宏任务，也可以是微任务，但是宏任务执行时机有着极大的不确定性，所以我们的 ```then()```中的参数在微任务中执行

</v-click>

---

async await

---
案例分析


---
vue的 nextTick
