## sigur

Sometimes callbacks can be quite annoying and frustrating. Everybody loves the `async`/`await` syntax popularized by C#, now Python has it, but JS would probably have it only in ES7.

That's a simple library to help with that problem for ES5 code.

## what does it do

can convert js code between vanilla ES5 callbacks code and a valid ES5 `async/await` inspired syntax.
can be useful for visualizing how existing callbacks-style code would look with `async/await`-like syntax
it's tuned for the codebase I am currently working with, so there are some pretty special cases/assumptions in it

## progress

* converts nested callbacks to series of `await`-s in `async`-annotated functions
* converts some cases of `async.each` to `await parallelEach(iterator => code)` code
* converts `async.waterfall` to series of `await` assignments

```javascript
function l(callback) {
    Person.find().exec(function (e, q) {
        callback(null, q);
    });
}

function cosn(q, callback) {
    callback([2]);
}

function loop(qs, callback) {
    async.each(qs, function (q, cb) {
        Person.generateStuff(q, cb);
    }, function (err) {
        console.debug('z');
        callback();
    });
}

async.waterfall([
    l,
    cosn,
    loop
]);
```

is currently converted to

```javascript
function l(callback) {
}

async function cosn(q) {
  return [2];
}
function loop(qs, callback) {
  await qs.parallelEach(q => {
    Person.generateStuff(q);
  });
  console.debug('z');
  return;
}
var q = await l();
var qs = await cosn(q);
await loop(qs);
```

##convert back?

using acorn-es7plugin it's possible to write a script that translates
back from the `async/await` output to equivalent to the original callback-based code

if that happens and one feels experimental enough, the usual workflow could be:

* open the file in a `async/await` mode (converted in the background by `sigur`)
* do your changes 
* `sigur` converts it back to callback-based ES5 

## technologies

`falafel` for ast rewriting, based on
`acorn` for analyzing and parsing ast