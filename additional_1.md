# 宏微任务真的就只有这些么？

<v-click at="1">

```js
Promise.resolve().then(() => {
  console.log(1);
  return Promise.resolve(3);
}).then(rt => {
  console.log(rt);
})
Promise.resolve().then(() => {
  console.log(2);
}).then(() => {
  console.log(4);
}).then(() => {
  console.log(5);
}).then(() => {
  console.log(6);
})
```

</v-click>

<v-click at="2">

之前提到，“微任务执行过程中，产生的新的微任务，v8引擎会将其丢进微任务队列里。”这是为了便于我们理解这么说，但是 [WHATWG](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint) 规范里明确说了这么一句：
```If the event loop's performing a microtask checkpoint is true, then return.```，在执行微任务（Run oldestMicrotask.）时还特地强调一句，微任务执行过程中可能会产生再次调用checkpoint的回调，这是为什么要设置检查点为 true 的原因

</v-click>