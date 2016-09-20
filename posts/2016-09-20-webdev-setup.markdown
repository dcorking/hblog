---
title: ReactJS, JSX, async/await, babel, webpack and getting it all working
tags: async, babel, webpack, npm, await
description: Getting a modern front end setup.
---

### Stranger in a strange land

I'm not primarily a front-end dev so getting everything setup and
configured for web development is particularly frustrating for me and
most blog posts, tutorials don't actually give explanations, just
copy-pasting blindly tons of little configs.

Here's my post that I'm using as reference for me and hopefully for
any other non-frontend developer that wants to use the latest and
greatest `JavaScript` like `fetch`, `async`, `await`, `React`, `JSX`.

### Getting started, compiling JavaScript to ...JavaScript

Because of fragmentation in implementations of the latest `JavaScript`
features, we'll use `babel` to compile our `JavaScript` using the
latest features to `JavaScript` that will work in `Chrome`, `Firefox`
and `Safari`.

`babel` has a concept of `plugins`. These are like features that you
can turn on during the compilation steps and are pretty
granular. Often you'll want a whole bunch of plugins together and that
is so common enough that `babel` has something called
`presets`. You can put that in separate `.babelrc` file, but I prefer
not having so much silly little config files, so you can also put it
in your package.json; example:

```javascript
 "babel": {
    "presets": [
      "react",
      "es2015",
      "stage-3"
    ],
    "plugins": [
        "transform-es2015-modules-commonjs",
        "transform-async-to-generator",
        "transform-runtime"
    ]
  }
```

These are the ones I'm using to compile `JSX`, use ES6 modules, and
`async`, `await`.

So when you invoke `babel`, it will look at the `package.json`, see
the `babel` field and turn on those features, so an example invocation
is:

```shell
$ babel lib --out-dir dist
```

which will compile all the code in the `lib` directory and output the
results in the `dist` directory. This process is the same for `node`.

### Bundling code
Now we have our legal `JavaScript` for today's browsers/node. We can bundle
up everything as a single `JavaScript` file using `webpack`. I
previously used `browserify` but like all things web, apparently its
not hot anymore. We can invoke it like so:

```shell
$ webpack --progress --colors dist/homepage.js bundle.js
```

where `bundle.js` is the name of the single output file that we'll
get. You can apparently do some kind of config file for webpack, yet
another config file, but this is enough for me right now. 

### Actual code/project with JSX

So let's say we have these two files, one is `homepage.jsx` and the
other is `button.jsx`. Note that I use a real example of `async`,
`await`, for a great explanation
see [here](https://zeit.co/blog/async-and-await), for `OCaml`
programmers, `await` is basically `>>=` or `let%lwt`.

This is `button.jsx`

```javascript
'use strict';
import React from 'react';

class Button extends React.Component {

  async do_request(e) {
    let query =
	'https://api.bitcoinaverage.com' +
	'/ticker/global/USD';
    let nonsense = "https://foo.bar";
    try {
      let pulled = await fetch(query);
      let body = await pulled.json();
      console.log(body);

      await fetch(nonsense);

    } catch (e) {
      console.log("Exception raised:", e);
      console.log('Logic continued');
    }
  }

  render () {
    let s = {color:'red'};
    return (
      <p style={s}
	 onClick={this.do_request.bind(this)}>
	Click Me
      </p>
    );
  }
};
// Remember to put wrap in {}
export {Button};
```

and `homepage.jsx`

```javascript
'use strict';

import React from 'react';
import ReactDOM from 'react-dom';
// REMEMBER to do {} since button.jsx doesn't do
// export default
import {Button} from './button';

class Page extends React.Component {
  render () {
    return (
      <div>
	Hello World
	<Button/>
      </div>
    );
  }
};

ReactDOM.render(<Page/>,
		document.getElementById('cont'));
```

So all this will be compiled correctly and turned into one `bundle.js`
which we can use in this `index.html`

```html
<!DOCTYPE html>
<meta charset="utf-8">
<body>
  <div id="cont"></div>
  <script src="bundle.js"></script>
</body>
```

and when we click the button we see this in the Chrome dev tools:

```
Object {24h_avg: 614.98, ask: 613.72, bid: 613.05, last: 613.56, timestamp: "Tue, 20 Sep 2016 20:09:30 -0000"…}
bundle.js:23085 GET https://foo.bar/ net::ERR_NAME_NOT_RESOLVED_callee$ @ bundle.js:23085tryCatch @ bundle.js:23242invoke @ bundle.js:23516prototype.(anonymous function) @ bundle.js:23275step @ bundle.js:23872(anonymous function) @ bundle.js:23883
bundle.js:23095 Exception raised: TypeError: Failed to fetch(…)
bundle.js:23096 Logic continued
```

Yay, things worked.

See the
repo
[here](https://github.com/fxfactorial/react-example-with-async-await-babel) for
the full `package.json`
