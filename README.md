[aa](https://www.npmjs.com/package/aa) - [async-await](https://www.npmjs.com/package/async-await)
====

  [aa](https://www.npmjs.com/package/aa) - [async-await](https://www.npmjs.com/package/async-await) library.<br/>
  co like library, go like channel, thunkify or promisefy wrap package.

  using ES6 (ES2015) generator function.

  compatible with co@3 and co@4.


INSTALL:
----

```bash
$ npm install aa --save
   or
$ npm install async-await --save
```

[![NPM](https://nodei.co/npm/aa.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/aa/)
[![NPM](https://nodei.co/npm-dl/aa.png?height=2)](https://nodei.co/npm/aa/)
[![NPM](https://nodei.co/npm/async-await.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/async-await/)
[![NPM](https://nodei.co/npm-dl/async-await.png?height=2)](https://nodei.co/npm/async-await/)


PREPARE:
----

```js
  var aa = require('aa');
  // or
  var aa = require('async-await');
```


USAGE:
----

### aa(generator or generator function) : returns promise (thunkified promise)

basic usage. <br>
you can `aa()` promises, generators, and generator functions.

```js
aa(function *() {
	// *** SEQUENTIAL EXECUTION ***

	// yield value returns itself
	yiled 1;
	yiled [1, 2, 3];
	yiled {x:1, y:2, z:3};

	// yield promise returns resolved value
	yield Promise.resolve(1);

	// or throws rejected error
	try { yield Promise.reject(new Error('expected')); }
	catch (e) { console.error('%s', e); }

	// *** PARALLEL EXECUTION ***

	// yield an array of promises waits all promises and returns resolved array
	yield [Promise.resolve(1), Promise.resolve(2)];

	// yield an object of promises waits all promises and returns resolved object
	yield {x: Promise.resolve(1), y: Promise.resolve(2)};

	// *** OTHERS AND COMBINED OPERATIONS ***

	// yield thunk
	// yield generator or generator function
	// yield channel for event stream
});
```


### aa.promisify(node style function) : function returns promise

`promisify()` converts node style function into a function returns promise. <br>
you can use `fs.exists()` and `child_process.exec()` also.

### aa.thunkify(node style function) : function returns thunk

`thunkify()` converts node style function into a thunkified function. <br>
you can use `fs.exists()` and `child_process.exec()` also.

### aa.Channel() : new channel for event stream

`Channel()` returns a new channel for event stream. <br>
use a channel for node style function as a callback. <br>
yield channel for wait it.  <br>


### yield : waits and returns resolved value.

you can `yield` promises, thunkified functions,
generators, generator functions,
primitive values, arrays, and objects. <br>


EXAMPLES:
----

### Example 1 sequential: [aa-readme-ex01-seq.js](examples/aa-readme-ex01-seq.js#readme)

```bash
$ node aa-readme-ex01-seq.js
```

```js
	var aa = require('aa');


	aa(main);


	function *main() {
		console.log('11:', yield asyncPromise(100, 11));
		console.log('12:', yield asyncThunk(100, 12));
		console.log('13:', yield asyncGenerator(100, 13));
		yield sub(20);
		yield sub(30);
	}


	function *sub(base) {
		console.log('%s: %s', base + 1, yield asyncPromise(100, base + 1));
		console.log('%s: %s', base + 2, yield asyncThunk(100, base + 2));
		console.log('%s: %s', base + 3, yield asyncGenerator(100, base + 3));
	}


	// asyncPromise(ms, arg) : promise
	function asyncPromise(ms, arg) {
		return new Promise(function (res, rej) {
			setTimeout(function () { res(arg); }, ms);
		});
	}


	// asyncThunk(ms, arg) : thunk
	function asyncThunk(ms, arg) {
		return function (cb) {
			setTimeout(function () { cb(null, arg); }, ms);
		};
	}


	// asyncGenerator(ms, arg) : generator
	function *asyncGenerator(ms, arg) {
		var chan = aa.Channel();
		setTimeout(function () { chan(arg); }, ms);
		return yield chan;
	}
```


### Example 2 parallel: [aa-readme-ex02-par.js](examples/aa-readme-ex02-par.js#readme)

```bash
$ node aa-readme-ex02-par.js
```

```js
	var aa = require('aa');


	aa(main);


	function *main() {
		console.log('[11, 12, 13]:', yield [
			asyncPromise(100, 11),
			asyncThunk(100, 12),
			asyncGenerator(100, 13)
		]);

		console.log('{x:11, y:12, z:13}:', yield {
			x: asyncPromise(100, 11),
			y: asyncThunk(100, 12),
			z: asyncGenerator(100, 13)
		});

		yield [sub(20), sub(30)];
	}


	function *sub(base) {
		console.log('%s: %s', base + 1, yield asyncPromise(100, base + 1));
		console.log('%s: %s', base + 2, yield asyncThunk(100, base + 2));
		console.log('%s: %s', base + 3, yield asyncGenerator(100, base + 3));
	}


	// asyncPromise(ms, arg) : promise
	function asyncPromise(ms, arg) {
		return new Promise(function (res, rej) {
			setTimeout(function () { res(arg); }, ms);
		});
	}


	// asyncThunk(ms, arg) : thunk
	function asyncThunk(ms, arg) {
		return function (cb) {
			setTimeout(function () { cb(null, arg); }, ms);
		};
	}


	// asyncGenerator(ms, arg) : generator
	function *asyncGenerator(ms, arg) {
		var chan = aa.Channel();
		setTimeout(function () { chan(arg); }, ms);
		return yield chan;
	}
```


### Example promisify: [aa-readme-ex11-promisify.js](examples/aa-readme-ex11-promisify.js#readme)

```bash
$ node aa-readme-ex11-promisify.js
```

```js
	var aa = require('aa');
	var promisify = aa.promisify;
	var asyncPromise = promisify(asyncCallback);


	aa(main);


	function *main() {
		console.log('11:', yield asyncPromise(100, 11));
		console.log('12:', yield asyncPromise(100, 12));
		console.log('13:', yield asyncPromise(100, 13));

		asyncPromise(100, 21)
		.then(function (val) {
			console.log('21:', val);
			return asyncPromise(100, 22);
		})
		.then(function (val) {
			console.log('22:', val);
			return asyncPromise(100, 23);
		})
		.then(function (val) {
			console.log('23:', val);
		});
	}


	// asyncCallback(ms, arg. cb) : node style normal callback
	// cb : function (err, val)
	function asyncCallback(ms, arg, cb) {
		setTimeout(function () { cb(null, arg); }, ms);
	}
```


### Example thunkify: [aa-readme-ex12-thunkify.js](examples/aa-readme-ex12-thunkify.js#readme)

```bash
$ node aa-readme-ex12-thunkify.js
```

```js
	var aa = require('aa');
	var thunkify = aa.thunkify;
	var asyncThunk = thunkify(asyncCallback);


	aa(main);


	function *main() {
		console.log('11:', yield asyncThunk(100, 11));
		console.log('12:', yield asyncThunk(100, 12));
		console.log('13:', yield asyncThunk(100, 13));

		asyncThunk(100, 21)
		(function (err, val) {
			console.log('21:', val);
			asyncThunk(100, 22)
			(function (err, val) {
				console.log('22:', val);
				asyncThunk(100, 23)
				(function (err, val) {
					console.log('23:', val);
				});
			});
		});
	}


	// asyncCallback(ms, arg. cb) : node style normal callback
	// cb : function (err, val)
	function asyncCallback(ms, arg, cb) {
		setTimeout(function () { cb(null, arg); }, ms);
	}
```


### Quick example collection: [aa-readme-example.js](examples/aa-readme-example.js#readme)

```bash
$ node aa-readme-example.js
```


```js
  var aa = require('aa');


  // sleep(ms, args,... cb) : node style normal callback
  function sleep(ms) {
    var args = [].slice.call(arguments, 1);
    setTimeout.apply(null, [args.pop(), ms, null].concat(args));
  }

  sleep(1000, function (err, val) { console.log('1000 ms OK'); });


  // delay(ms, args,...)(cb) : thunk
  function delay(ms) {
    var args = [].slice.call(arguments);
    return function (cb) {
      sleep.apply(null, args.concat(cb));
    };
  }
  // var delay = aa.thunkify(sleep);

  delay(1100)(
    function (err, val) { console.log('1100 ms OK'); }
  );


  // aa.promisify(fn)   : returns wrapped function a.k.a thunkify and promisify
  // wait(ms, args,...) : returns promise & thunk
  var wait = aa.promisify(sleep);

  // wait() : as a thunk
  wait(1200)(
    function (err, val) { console.log('1200 ms OK'); }
  );

  // wait() : as a promise
  wait(1300).then(
    function (val) { console.log('1300 ms OK'); },
    function (err) { console.log('1300 ms NG', err); }
  ).catch(
    function (err) { console.log('1300 ms NG2', err); }
  );


  // aa(generator) : returns promise & thunk
  aa(function *() {

    yield 1;                    // primitive value
    yield [1, 2, 3];            // array
    yield {x:1, y:2, z:3};      // object


    // wait for promise
    yield Promise.resolve(2);


    // wait for thunk
    yield delay(800);


    // wait for promise or thunk
    yield wait(800);


    console.log('0:', yield wait(300, 0));
    console.log('1:', yield wait(300, 1));


    // yield Promise.all([])
    console.log('[1, 2, 3]:',
      yield Promise.all([wait(200, 1), wait(300, 2), wait(100, 3)]));


    // yield [] -> like Promise.all([]) !
    console.log('[4, 5, 6]:',
      yield [wait(200, 4), wait(300, 5), wait(100, 6)]);


    // yield {} -> like Promise.all({}) !?
    console.log('{x:7, y:8, z:9}:',
      yield {x:wait(200, 7), y:wait(300, 8), z:wait(100, 9)});


    // make channel for sync - fork and join
    var chan = aa.Channel();

    sleep(300, 20, chan);   // send value to channel : fork or spread
    sleep(200, 10, chan);   // send value to channel : fork or spread
    var a = yield chan;     // recv value from channel : join or sync
    var b = yield chan;     // recv value from channel : join or sync
    console.log('10 20:', a, b);


    // fork thread -  make new thread and start
    aa(function *() {
      yield wait(200);      // wait 200 ms
      return 200;
    })(chan);               // send 200 to channel : join or sync

    // fork thread -  make new thread and start
    aa(function *() {
      yield wait(100);      // wait 100 ms
      return 100;
    })(chan);               // send 100 to channel : join or sync

    // fork thread -  make new thread and start
    aa(function *() {
      yield wait(300);      // wait 300
      return 300;
    })(chan);               // send 300 to channel : join or sync

    // join threads - sync threads
    var x = yield chan;     // wait & recv first  value from channel
    var y = yield chan;     // wait & recv second value from channel
    var z = yield chan;     // wait & recv third  value from channel
    console.log('top 3 winners: 100 200 300:', x, y, z);


    // communicate with channels
    var chan1 = aa.Channel(), chan2 = aa.Channel();

    // thread 1: send to chan1, recv from chan2
    aa(function *() {
      sleep(100, 111, chan1);
      console.log('222:', yield chan2);
      sleep(100, 333, chan1);
      console.log('444:', yield chan2);
      sleep(100, 555, chan1);
      return 666;
    })(chan);

    // thread 1: recv from chan1, send to chan2
    aa(function *() {
      console.log('111:', yield chan1);
      sleep(100, 222, chan2);
      console.log('333:', yield chan1);
      sleep(100, 444, chan2);
      console.log('555:', yield chan1);
      return 777;
    })(chan);
    console.log('666 777:', yield chan, yield chan);

    return 11;
  })
  .then(
    function (val) {
      console.log('11 val:', val);
      return wait(100, 22); },
    function (err) {
      console.log('11 err:', err);
      return wait(100, 22); }
  )
  (function (err, val) {
      console.log('22 val:', val, err ? 'err:' + err : '');
      return wait(100, 33); })
  (function (err, val) {
      console.log('33 val:', val, err ? 'err:' + err : '');
      return wait(100, 44); })
  .then(
    function (val) {
      console.log('44 val:', val);
      return wait(100, 55); },
    function (err) {
      console.log('44 err:', err);
      return wait(100, 55); }
  )
  .catch(
    function (err) {
      console.log('55 err:', err);
      return wait(100, 66); }
  );
```


LICENSE:
----

  MIT
