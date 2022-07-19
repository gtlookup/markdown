# 状态

1）待定（pending）：初始化状态，既没兑现也没拒绝

2）已兑现（fulfilled）：操作成功完成

3）已拒绝（rejected）：操作失败

```js
new Promise((y, n) => {
    y('haha'); // then haha，已兑现（fulfilled）
    n('haha'); // catch haha，已拒绝（rejected）
}).then(x => console.log('then', x)).catch(x => console.log('catch', x));
```

# 静态方法

## 1. all

所有都成功，则 then；否则 catch

```js
Promise.all([
    new Promise((y, n) => y('a')),
    new Promise((y, n) => n('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：catch b

Promise.all([
    new Promise((y, n) => y('a')),
    new Promise((y, n) => y('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：then ["a", "b"]
```

## 2. allSettled

返回每个 promise 的结果

```js
Promise.allSettled([
    new Promise((y, n) => y('a')),
    new Promise((y, n) => n('b'))
]).then(x => console.log('then', x));
// 结果：[{status: "fulfilled", value: "a"}, {status: "rejected", reason: "b"}]
// fulfilled => 成功，回返 value
// rejected => 失败，返回 reason
```

## 3. any

从前往后数，取第1个成功的

```js
Promise.any([
    new Promise((y, n) => n('a')),
    new Promise((y, n) => y('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：then b
```

```js
Promise.any([
    new Promise((y, n) => n('a')),
    new Promise((y, n) => n('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：catch AggregateError: All promises were rejected
```

## 4. race

返回第1个完成的，成功就 then，失败就 catch

```js
Promise.race([
    new Promise((y, n) => y('a')),
    new Promise((y, n) => n('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：then a
```

```js
Promise.race([
    new Promise((y, n) => n('a')),
    new Promise((y, n) => y('b'))
]).then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：catch a
```

## 5. reject

返回一个失败的 promise

```js
Promise.reject("a")
    .then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：catch a
```

## 6. resolve

返回一个成功的 promise

```js
Promise.resolve("a")
    .then(x => console.log('then', x)).catch(x => console.log('catch', x));
// 结果：then a
```



