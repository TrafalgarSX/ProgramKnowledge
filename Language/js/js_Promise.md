### Promise
---
Promise 对象是 JavaScript 中用于处理异步操作的一种机制。**它代表了一个异步操作的最终结果，可以用于处理异步操作的成功或失败。**

Promise 对象有三种状态：
- pending（进行中）、
- fulfilled（已成功)
- rejected（已失败）。

当一个 Promise 对象被创建时，它会处于 pending 状态，表示异步操作正在进行中。当异步操作成功完成时，Promise 对象会变为 fulfilled 状态，表示异步操作已成功，并将结果传递给后续的处理函数。当异步操作发生错误或失败时，Promise 对象会变为 rejected 状态，并将错误原因传递给后续的错误处理函数。

Promise 对象通过**链式调用的方式来处理异步操作的结果**。
你可以使用 `.then()` 方法来注册成功处理函数，用于处理异步操作成功的情况。
你也可以使用 `.catch()` 方法来注册错误处理函数，用于处理异步操作失败的情况。
此外，还可以使用 `.finally()` 方法来注册无论异步操作成功或失败都会执行的回调函数。

Promise 对象的优点是可以解决回调地狱（callback hell）的问题，使异步操作的代码更加清晰和易于理解。它提供了一种更优雅的方式来处理异步操作，并可以通过链式调用来串联多个异步操作。

### async await
---
- `async`：用于声明一个函数是异步函数，**返回一个 Promise 对象**。异步函数内部可以包含 `await` 表达式。

- `await`：只能在异步函数内部使用，**用于暂停函数的执行，等待一个 Promise 对象的解析结果**。当 `await` 后面的 Promise 对象成功解析时，**会返回解析结果**，然后函数继续执行。

一个 `async` 函数会返回一个 Promise 对象。

当你在定义一个 `async` 函数时，它会自动封装成一个 Promise 对象。这意味着当你调用该函数时，它会返回一个 Promise 对象，而不是直接返回函数的结果。

`async` 函数内部可以包含 `await` 关键字，用于等待一个异步操作的结果。**当遇到 `await` 关键字时，`async` 函数会暂停执行，直到等待的异步操作完成并返回结果**。然后，`async` 函数会继续执行，并将异步操作的结果作为 Promise 对象的解析值返回。

如果在 `async` 函数中没有显式地返回一个值，它会隐式地返回一个解析值为 `undefined` 的 Promise 对象。

以下是一个示例，展示了 `async` 函数返回一个 Promise 对象的情况：

```javascript
javascriptasync function fetchData() {
  // 异步操作
  return 'Data fetched successfully';
}

const promise = fetchData();
console.log(promise); // Promise {<fulfilled>: 'Data fetched successfully'}
```

在上面的示例中，`fetchData` 函数返回一个字符串，并自动被封装成一个 Promise 对象。你可以通过 `then()` 方法链式调用来处理该 Promise 对象的结果。

### 如何解析 Promise 对象
---
1. 使用 `await`：在异步函数内部，可以使用 `await` 关键字等待 `Promise` 对象的解析结果，并将解析的值返回。

2. 使用 `.then()`：可以使用 `Promise` 对象的 `.then()` 方法来注册解析成功时的回调函数，并获取解析的值。
```javascript
javascript
async function getData() {
  const result = await someAsyncOperation();
  return result;
}

getData().then(value => {
  console.log(value);
});
```

3. 使用 `Promise.all()`：如果我们有**多个异步操作**，并且希望同时等待它们全部完成，我们可以使用 `Promise.all()` 方法。这个方法接受一个包含多个 Promise 对象的数组，并返回一个新的 Promise 对象。这个新的 Promise 对象在所有的 Promise 对象都解析成功时才会解析成功，并返回一个包含所有解析值的数组。
```javascript
javascript
async function fetchData() {
  const promise1 = fetch('https://api.example.com/data1');
  const promise2 = fetch('https://api.example.com/data2');
  const promise3 = fetch('https://api.example.com/data3');

  const results = await Promise.all([promise1, promise2, promise3]);

  return results;
}

fetchData().then(data => {
  console.log(data); // 包含所有解析值的数组
});
```
