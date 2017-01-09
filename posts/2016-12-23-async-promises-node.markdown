---
title: node's async, await, Promises. 
tags: async, lwt, await, async, promises
description: Solidifying understand of Promises.
---

While studying for interviews I'm talking breaks by working
on [silicondzor.com](https://github.com/fxfactorial/silicondzor)

This is giving me some real web dev experience and I'm using the
latest and greatest features of `JavaScript` and `node`.

One thing that I'm really liking in modern `JavaScript` is the `async`
story, it reminds me a lot of `OCaml` and both are somewhat
converging, i.e `ES6`'s introduction of Promises which are basically
`'a Lwt.t`, and Lwt renaming `Lwt.t` into Promises in `2.7.0`.

Here's like my conceptual cheatsheet about using `Promises` from
scratch, converting an callback API into a `Promises` API and then
taking it to the next step with `async`, `await`.

# Basic callback APIs

`node` uses an event based programming paradigm and this is reflected
in basically all server side code, ie the trailing callback argument
to asynchronously call once the task completes.

Let's simulate it with this function 

```javascript
// Plain CB based API
const test_func = (item, cb) => {
  setTimeout(() => {
    if (item < 5) cb(null, 'Success');
    else cb(new Error('Oops'), null);
  }, 3000);
};

test_func(3, (err, success) => console.log(success));
```

This models typical `node` code, the callback being called `three`
seconds after execution of `test_func`

This seems pretty fine for this example but once you have logic that
is dependent on the success of one callback API after another then
you start to have a difficult to reason about triangle of callbacks on
the screen.

# Promises 

`ES6` introduces `Promises` and these are quite convienent, think of a
Promise as being a box which will have a value in it sometime later. 

We can turn any callback based API into a Promises based one. 

```javascript
// Promises based
const test_func_promise = item => {
  return new Promise((accept, reject) => {
    test_func(item, (err, success) => {
      if (err) reject(err);
      else accept(success);
    });
  });
};

test_func_promise(2)
.then(item => { console.log('Sucess!!', item); })
.catch(err => console.error(err));
```

When you create a new `Promise` you need to provide it with two
function, one to be called when there's a success and when the Promise
should fail. Hence when calling `test_func_promise` returns a Promise,
not the value `'Success'`. Also if you use the plain Promises approach
be sure to include a `.catch` call.

This is nicer than the original callback API approach, but now all
your success or failure logic and everything following that
effectively has to be all in either `.then` or `.catch` and that's a
bit awkward still.

# async/await

`ES7` introduced `async`, `await` where I basically think of the
latter as `>>=` and `async` as sort of like `>|=`. With `async`,
`await` we can call `Promises` based code as if it was synchronous
code.

```javascript
const test_func_async_example = async (item) => {
  try {
    const result = await test_func_promise(item);
    console.log(`Yay: ${result}`);
  } catch(e) {
    console.error(`Messed up: ${e}`);
  }
};

test_func_async_example(6);
test_func_async_example(3);
```

First notice how whatever function we use `await` inside of, we need
to tag that function with `async` and now this function returns a
`Promise`

Now when we call `test_func_promises` with `await` prefixed, then we
get the value that the `Promise` resolved. Also whatever the `reject`
condition of the `Promise` was then becomes the exception of the
`catch` following the `await` usage.

# Summary

`async`, `await` are very powerful features of `ES7`, `Promises` are
from `ES6`. Not every `JavaScript` engine supports `ES7` so you'll
have to use `babel` to compile your `async`, `await` to runnable
code; basically turns the `async`, `await` to generators (also an
amazing topic).

On newer versions of node, I'm using `v7.3.0`, you can turn on
features from the future with command line arguments. Check all of
them with:

```
$ node --v8-options | less
```

I'm turning on native `async`, `await` with `--harmony_async_await`,
so my final invocation is: 

```
$ node --harmony_async_await test.js
```

You can get all the code as one
script
[here](https://gist.github.com/fxfactorial/60edd7c3c8948754d21dbc9f517e37ee)
