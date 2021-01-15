---
layout: post
title: "JavaScript Promises with Async / Await and Loops"
description: "An example driven guide on using promises with async / await and integrating into loops"
date: 2021-01-14
tags: [promises,async/await,javascript,loops,settimeout]
comments: true
share: true
---
# JavaScript Promises and Async / Await with Loops

This article will be an example driven guide on writing promises, consuming them through `async / await` code, and using them in loops.

As a note, this article assumes that you are already familiar with concepts such as [Blocking vs Non-Blocking]() code in JavaScript and have maybe even seen (or written) a promise before.

If you are interested in more of a deep dive on promises, be sure to check out Mozilla's [official docs on promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) on the MDN website.

## The Example We Will Build Up To

For those who want to skip straight to the conclusion, here is the example that we are building up to:

```javascript
// promisified 'setTimeout' function
function delay(delayedFunction, time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      try {
        resolve(delayedFunction());
      } catch (error) {
        reject(error.message);
      }
    }, time);
  });
}

// top-level async declaration
async function main() {
  // await using a for...of loop
  const delaysInMs = [3000, 2000, 1000];

  for (const time of delaysInMs) {
    console.log(await delay(() => `print after ${time / 1000} second(s)`, time));
  }

  // await using a while loop
  let count = 0;
  const max = 100;

  while (count < max) {
    const result = await delay(() => `${++count} of ${max} lines...`, 1000);
    console.log(result);
  }
}

main();
```


## A Classic Way of Delaying Code Execution

A great place to illustrate the benefits of `Promises` in JavaScript is to fiddle with a classic built-in function called `setTimeout`.

This function takes two parameters (the first being a function that you would like to execute, and the second being time in milliseconds) and delays the execution of the function by that amount of time:

```javascript
setTimeout(inputFunction, timeInMilliseconds);
```

It is also common place for programmers to use an anonymous function as the `inputFunction`:

```javascript
console.log('1 | print immediately');
setTimeout(() => {
  console.log('2 | print after one second');
}, 1000);
```


The following would output:

```txt
1 | print immediately
2 | print after one second
```

Where line `2` in fact would output into the console after one **second**.

Now lets say we wanted to print a third line which prints exactly **one second** after line `2`. We could go about it in two naive ways:

In one way, we manually change the time value to `2000` in the second invocation of `setTimeout` to simulate line `3` executing one second after line `2`:

```javascript
console.log('1 | print immediately');
setTimeout(() => {
  console.log('2 | print after one second');
}, 1000);
setTimeout(() => {
  console.log('3 | print one second after line 2');
}, 2000);
```


In another way, we nest the first `setTimeout` with another `setTimeout` which has the same time value:

```javascript
console.log('1 | print immediately');
setTimeout(() => {
  console.log('2 | print after one second');
  setTimeout(() => {
    console.log('3 | print one second after line 2');
  }, 1000);
}, 1000);
```


Now you could imagine how messy this gets if we wanted to print 100 lines, all one second after another.

Lets _promisify_ our beloved `setTimeout` function and you'll see how printing 100 lines one second after another won't be all that difficult.

## Declaring A Promise

A promise is not accessed or created using a _reserved word_, rather it is an object that must be instantiated using the built-in `Promise` class. In the case of promisifying our `setTimeout` function, we can declare a new function called `delay` like so:

```javascript
function delay(delayedFunction, time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(delayedFunction());
    }, time);
  });
}
```


Here our new `delay` function returns a `Promise` object. The promise takes a `function` as its first parameter and captures the two values `resolve` and `reject` inside of it (which are also functions as we shall explore below).

So far we have kepts things simple, but lets assume that we execute code that can crash or fail, or throw some kind of error in our `delayedFunction`, lets upgrade our example with some basic error handling and add `try / catch` blocks. This will let us utilize `reject(...)`:

```javascript
function delay(delayedFunction, time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      try {
        resolve(delayedFunction());
      } catch (error) {
        reject(error.message);
      }
    }, time);
  });
}
```


Now, we have a promise based `delay` function which we can feed functions into. When we invoke `delay`, we will followup by invoking a method made available by our promise object called `.then(...)`, lets try it out:

```javascript
console.log('1 | print immediately');
delay(() => '2 | print after one second', 1000)
  .then(result => console.log(result));
```


Similar as in our previous example, this also outputs:

```txt
1 | print immediately
2 | print me after one second
```

With the exact timing that the logs imply.

The `.then(...)` method runs using the **SUCCESSFULLY** returned results of the `delayedFunction`. We can see them being fed into `resolve(...)`. We also have another method called `.catch(...)` that we can use to handle **rejected** promises.

```javascript
console.log('1 | print immediately');
delay(() => {
  throw new Error('This is a fake error.');
  return '2 | print after one second';
}, 1000)
  .then(result => console.log(result))
  .catch(error => console.log(error));
```


Now this will output:

```
1 | print immediately
This is a fake error
```

Here we are intentionally throwing a new `Error` with an error message of `This is a fake error.` As you can see, we are able to _catch_ it perfectly.

This still doesn't fully address our problem though. If we wanted to execute a bunch of `delay` functions one after the other, we still have to deal with the following:

```
console.log('1 | print immediately');
delay(() => '2 | print me after one second', 1000)
  .then(result => {
    console.log(result);
    delay(() => '3 | print me one second after line 2', 1000)
      .then(result => {
        console.log(result);
        /* ...CALLBACK HELL... */
      });
  });
```


This becomes really difficult to read as our task(s) grow in complexity (and is also known as _**CALLBACK HELL**_). Here is where `async / await` comes in.

## Async / Await

Long story short, if something returns a `Promise` object, you can use `async / await` with it and avoid callback hell. Let's transform our previous example to illustrate the benefits.

First we have to declare a **top-level async function**, what this means is that we are letting JavaScript know that during the execution of a code block, we want to use the `await` reserved word inside it.

```javascript
function delay(delayedFunction, time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      try {
        resolve(delayedFunction());
      } catch (error) {
        reject(error.message);
      }
    }, time);
  });
}

async function main() {
  /* Now we can use 'await' in here with our `delay` function. */
}

main();
```


Now lets use `await` to invoke our delay function in an appropriate fashion:

```javascript
async function main() {
  console.log('1 | print immediately');
  console.log(await delay(() => '2 | print after one second', 1000));
}

main();
```


And we get our expected result of:

```txt
1 | print immediately
2 | print after one second
```

Now lets add quite a few more lines:

```javascript
async function main() {
  console.log('1 | print immediately');
  console.log(await delay(() => '2 | print after one second', 1000));
  console.log(await delay(() => '3 | print one second after line 2', 1000));
  console.log(await delay(() => '4 | print one second after line 3', 1000));
  console.log(await delay(() => '5 | print one second after line 4', 1000));
}

main();
```


And we get:

![](https://res.cloudinary.com/dvivnklwq/image/upload/v1610634016/w0fwsnr4k4cfy7uzsqkv.gif)

What `await` is doing is replacing

```javascript
delay(() => '[RETURN VALUE]', 1000)
  .then(result => {
    console.log(result); // [RETURN VALUE] in result
  });
```


with

```javascript
const result = await delay(() => '[RETURN VALUE]', 1000);
console.log(result); // [RETURN VALUE] in result
```


So we can transform the following:

```javascript
delay(() => '[RETURN VALUE 1]', 1000)
  .then(result => {
    console.log(result); // [RETURN VALUE 1] in result
    delay(() => '[RETURN VALUE 2]', 1000)
      .then(result => {
        console.log(result); // [RETURN VALUE 2] in result
      });
  });
```


into

```javascript
let result;

result = await delay(() => '[RETURN VALUE 1]', 1000);
console.log(result); // [RETURN VALUE 1] in result

result = await delay(() => '[RETURN VALUE 2]', 1000);
console.log(result); // [RETURN VALUE 2] in result
```


Much cleaner isn't it?

## Async / Await in Loops

There may be multiple ways of combining `async / await` with iteration, in this article we will explore just two basic ways:

### Using a `for loop`

You can use `await` very easily inside of a `for ... of` loop:

```javascript
async function main() {
  const delaysInMs = [3000, 2000, 1000];
  
  for (const time of delaysInMs) {
    const result = await delay(() => `print after ${time / 1000} second(s)`, time);
    console.log(result);
  }
}

main();
```


### Using a `while loop`

You can also use a simple `while` loop:

```javascript
async function main() {
  let count = 0;
  const max = 5;

  while (count < max) {
    const result = await delay(() => `${++count} of ${max} lines...`, 1000);
    console.log(result);
  }
}

main();
```


And there you have it. Thanks for reading and happy coding!
