<!-- This EXTRA_DOCS.md file is automatically generated from source code and files in the /markdown directory. Please DO NOT send pull requests to directly modify this file. Instead, edit the JSDoc comments in source code or the md files in /markdown or the md.ejs files in /tools. -->

## Extras

- [`buffer`](#buffer) (operator)
- [`concat`](#concat) (factory)
- [`debounce`](#debounce) (operator)
- [`delay`](#delay) (operator)
- [`dropRepeats`](#dropRepeats) (operator)
- [`dropUntil`](#dropUntil) (operator)
- [`flattenConcurrently`](#flattenConcurrently) (operator)
- [`flattenSequentially`](#flattenSequentially) (operator)
- [`fromDiagram`](#fromDiagram) (factory)
- [`pairwise`](#pairwise) (operator)
- [`sampleCombine`](#sampleCombine) (operator)
- [`split`](#split) (operator)
- [`throttle`](#throttle) (operator)
- [`tween`](#tween) (factory)

# How to use extras

The following are standalone stream operators and stream factories that may be separately imported and utilized in your project.
To use an extra operator (e.g. `delay`), import it as such:

```js
import xs from 'xstream'
import delay from 'xstream/extra/delay'

const inputStream = xs.of(1, 2, 3, 4)
const outputStream = inputStream.compose(delay(500))
```

To import and use an extra factory (e.g. `fromEvent`), import it as such:

```js
import fromEvent from 'xstream/extra/fromEvent'

const clickStream = fromEvent(document, 'click')
```

- - -

### <a id="buffer"></a> `buffer(separator)`

Buffers a stream using a separator stream. Returns a stream that emits
arrays.

Marble diagram:

```text
--1--2--3--4--5--6--7--8--9|
buffer( -a---------b---------c| )
---------[1,2,3]---[4,5,6]---[7,8,9]|
```

Example:

```js
import buffer from 'xstream/extra/buffer'

const source = xs.periodic(50).take(10);
const separator = xs.periodic(170).take(3);
const buffered = source.compose(buffer(separator));

buffered.addListener({
  next: arr => console.log(arr),
  error: err => console.error(err)
});
```

```text
> [0, 1, 2]
> [3, 4, 5]
> [6, 7, 8]
```

#### Arguments:

- `separator: Stream` Some other stream that is used to know when to split the output stream.

#### Returns:  Stream 

- - -

### <a id="concat"></a> `concat(stream1, stream2)`

Puts one stream after the other. *concat* is a factory that takes multiple
streams as arguments, and starts the `n+1`-th stream only when the `n`-th
stream has completed. It concatenates those streams together.

Marble diagram:

```text
--1--2---3---4-|
...............--a-b-c--d-|
          concat
--1--2---3---4---a-b-c--d-|
```

Example:

```js
import concat from 'xstream/extra/concat'

const streamA = xs.of('a', 'b', 'c')
const streamB = xs.of(10, 20, 30)
const streamC = xs.of('X', 'Y', 'Z')

const outputStream = concat(streamA, streamB, streamC)

outputStream.addListener({
  next: (x) => console.log(x),
  error: (err) => console.error(err),
  complete: () => console.log('concat completed'),
})
```

#### Arguments:

- `stream1: Stream` A stream to concatenate together with other streams.
- `stream2: Stream` A stream to concatenate together with other streams. Two or more streams may be given as arguments.

#### Returns:  Stream 

- - -

### <a id="debounce"></a> `debounce(period)`

Delays events until a certain amount of silence has passed. If that timespan
of silence is not met the event is dropped.

Marble diagram:

```text
--1----2--3--4----5|
    debounce(60)
-----1----------4--|
```

Example:

```js
import fromDiagram from 'xstream/extra/fromDiagram'
import debounce from 'xstream/extra/debounce'

const stream = fromDiagram('--1----2--3--4----5|')
 .compose(debounce(60))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> 1
> 4
> completed
```

#### Arguments:

- `period: number` The amount of silence required in milliseconds.

#### Returns:  Stream 

- - -

### <a id="delay"></a> `delay(period)`

Delays periodic events by a given time period.

Marble diagram:

```text
1----2--3--4----5|
    delay(60)
---1----2--3--4----5|
```

Example:

```js
import fromDiagram from 'xstream/extra/fromDiagram'
import delay from 'xstream/extra/delay'

const stream = fromDiagram('1----2--3--4----5|')
 .compose(delay(60))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> 1  (after 60 ms)
> 2  (after 160 ms)
> 3  (after 220 ms)
> 4  (after 280 ms)
> 5  (after 380 ms)
> completed
```

#### Arguments:

- `period: number` The amount of silence required in milliseconds.

#### Returns:  Stream 

- - -

### <a id="dropRepeats"></a> `dropRepeats(isEqual)`

Drops consecutive duplicate values in a stream.

Marble diagram:

```text
--1--2--1--1--1--2--3--4--3--3|
    dropRepeats
--1--2--1--------2--3--4--3---|
```

Example:

```js
import dropRepeats from 'xstream/extra/dropRepeats'

const stream = xs.of(1, 2, 1, 1, 1, 2, 3, 4, 3, 3)
  .compose(dropRepeats())

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> 1
> 2
> 1
> 2
> 3
> 4
> 3
> completed
```

Example with a custom isEqual function:

```js
import dropRepeats from 'xstream/extra/dropRepeats'

const stream = xs.of('a', 'b', 'a', 'A', 'B', 'b')
  .compose(dropRepeats((x, y) => x.toLowerCase() === y.toLowerCase()))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> a
> b
> a
> B
> completed
```

#### Arguments:

- `isEqual: Function` An optional function of type `(x: T, y: T) => boolean` that takes an event from the input stream and
checks if it is equal to previous event, by returning a boolean.

#### Returns:  Stream 

- - -

### <a id="dropUntil"></a> `dropUntil(other)`

Starts emitting the input stream when another stream emits a next event. The
output stream will complete if/when the other stream completes.

Marble diagram:

```text
---1---2-----3--4----5----6---
  dropUntil( --------a--b--| )
---------------------5----6|
```

Example:

```js
import dropUntil from 'xstream/extra/dropUntil'

const other = xs.periodic(220).take(1)

const stream = xs.periodic(50)
  .take(6)
  .compose(dropUntil(other))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> 4
> 5
> completed
```

#### Arguments:

#### Arguments:

- `other: Stream` Some other stream that is used to know when should the output stream of this operator start emitting.

#### Returns:  Stream 

- - -

### <a id="flattenConcurrently"></a> `flattenConcurrently()`

Flattens a "stream of streams", handling multiple concurrent nested streams
simultaneously.

If the input stream is a stream that emits streams, then this operator will
return an output stream which is a flat stream: emits regular events. The
flattening happens concurrently. It works like this: when the input stream
emits a nested stream, *flattenConcurrently* will start imitating that
nested one. When the next nested stream is emitted on the input stream,
*flattenConcurrently* will also imitate that new one, but will continue to
imitate the previous nested streams as well.

Marble diagram:

```text
--+--------+---------------
  \        \
   \       ----1----2---3--
   --a--b----c----d--------
    flattenConcurrently
-----a--b----c-1--d-2---3--
```

#### Returns:  Stream 

- - -

### <a id="flattenSequentially"></a> `flattenSequentially()`

Flattens a "stream of streams", handling only one nested stream at a time,
with no concurrency, but does not drop nested streams like `flatten` does.

If the input stream is a stream that emits streams, then this operator will
return an output stream which is a flat stream: emits regular events. The
flattening happens sequentially and without concurrency. It works like this:
when the input stream emits a nested stream, *flattenSequentially* will start
imitating that nested one. When the next nested stream is emitted on the
input stream, *flattenSequentially* will keep that in a buffer, and only
start imitating it once the previous nested stream completes.

In essence, `flattenSequentially` concatenates all nested streams.

Marble diagram:

```text
--+--------+-------------------------
  \        \
   \       ----1----2---3--|
   --a--b----c----d--|
         flattenSequentially
-----a--b----c----d------1----2---3--
```

#### Returns:  Stream 

- - -

### <a id="fromDiagram"></a> `fromDiagram(diagram, options)`

Creates a real stream out of an ASCII drawing of a stream. Each string
character represents an amount of time passed (by default, 20 milliseconds).
`-` characters represent nothing special, `|` is a symbol to mark the
completion of the stream, `#` is an error on the stream, and any other
character is a "next" event.

Example:

```js
import fromDiagram from 'xstream/extra/fromDiagram'

const stream = fromDiagram('--a--b---c-d--|')

stream.addListener({
  next: (x) => console.log(x),
  error: (err) => console.error(err),
  complete: () => console.log('concat completed'),
})
```

The character `a` represent emission of the event `'a'`, a string. If you
want to emit something else than a string, you need to provide those values
in the options argument.

Example:

```js
import fromDiagram from 'xstream/extra/fromDiagram'

const stream = fromDiagram('--a--b---c-d--|', {
  values: {a: 10, b: 20, c: 30, d: 40}
})

stream.addListener({
  next: (x) => console.log(x),
  error: (err) => console.error(err),
  complete: () => console.log('concat completed'),
})
```

That way, the stream will emit the numbers 10, 20, 30, 40. The `options`
argument may also take `timeUnit`, a number to configure how many
milliseconds does each represents, and `errorValue`, a value to send out as
the error which `#` represents.

#### Arguments:

- `diagram: string` A string representing a timeline of values, error, or complete notifications that should happen on the output stream.
- `options` An options object that allows you to configure some additional details of the creation of the stream.

#### Returns:  Stream 

- - -

### <a id="pairwise"></a> `pairwise()`

Group consecutive pairs of events as arrays. Each array has two items.

Marble diagram:

```text
---1---2-----3-----4-----5--------|
      pairwise
-------[1,2]-[2,3]-[3,4]-[4,5]----|
```

Example:

```js
import pairwise from 'xstream/extra/pairwise'

const stream = xs.of(1, 2, 3, 4, 5, 6).compose(pairwise)

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> [1,2]
> [2,3]
> [3,4]
> [4,5]
> [5,6]
> completed
```

#### Returns:  Stream 

- - -

### <a id="sampleCombine"></a> `sampleCombine(streams)`

Combines a source stream with multiple other streams. The result stream
will emit the latest events from all input streams, but only when the
source stream emits.

If the source, or any input stream, throws an error, the result stream
will propagate the error. If any input streams end, their final emitted
value will remain in the array of any subsequent events from the result
stream.

The result stream will only complete upon completion of the source stream.

Marble diagram:

```text
--1----2-----3--------4--- (source)
----a-----b-----c--d------ (other)
     sampleCombine
-------2a----3b-------4d--
```

Examples:

```js
import sampleCombine from 'xstream/extra/sampleCombine'
import xs from 'xstream'

const sampler = xs.periodic(1000).take(3)
const other = xs.periodic(100)

const stream = sampler.compose(sampleCombine(other))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> [0, 8]
> [1, 18]
> [2, 28]
```

```js
import sampleCombine from 'xstream/extra/sampleCombine'
import xs from 'xstream'

const sampler = xs.periodic(1000).take(3)
const other = xs.periodic(100).take(2)

const stream = sampler.compose(sampleCombine(other))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> [0, 1]
> [1, 1]
> [2, 1]
```

#### Arguments:

- `streams: Stream` One or more streams to combine with the sampler stream.

#### Returns:  Stream 

- - -

### <a id="split"></a> `split(separator)`

Splits a stream using a separator stream. Returns a stream that emits
streams.

Marble diagram:

```text
--1--2--3--4--5--6--7--8--9|
 split( --a----b--- )
---------------------------|
  :        :    :
  1--2--3-|:    :
           4--5|:
                -6--7--8--9|
```

Example:

```js
import split from 'xstream/extra/split'
import concat from 'xstream/extra/concat'

const source = xs.periodic(50).take(10)
const separator = concat(xs.periodic(167).take(2), xs.never())
const result = source.compose(split(separator))

result.addListener({
  next: stream => {
    stream.addListener({
      next: i => console.log(i),
      error: err => console.error(err),
      complete: () => console.log('inner completed')
    })
  },
  error: err => console.error(err),
  complete: () => console.log('outer completed')
})
```

```text
> 0
> 1
> 2
> inner completed
> 3
> 4
> 5
> inner completed
> 6
> 7
> 8
> 9
> inner completed
> outer completed
```

#### Arguments:

- `separator: Stream` Some other stream that is used to know when to split the output stream.

#### Returns:  Stream 

- - -

### <a id="throttle"></a> `throttle(period)`

Emits event and drops subsequent events until a certain amount of silence has passed.

Marble diagram:

```text
--1-2-----3--4----5|
    throttle(60)
--1-------3-------5-|
```

Example:

```js
import fromDiagram from 'xstream/extra/fromDiagram'
import throttle from 'xstream/extra/throttle'

const stream = fromDiagram('--1-2-----3--4----5|')
 .compose(throttle(60))

stream.addListener({
  next: i => console.log(i),
  error: err => console.error(err),
  complete: () => console.log('completed')
})
```

```text
> 1
> 3
> 5
> completed
```

#### Arguments:

- `period: number` The amount of silence required in milliseconds.

#### Returns:  Stream 

- - -

### <a id="tween"></a> `tween(config)`

Creates a stream of numbers emitted in a quick burst, following a numeric
function like sine or elastic or quadratic. tween() is meant for creating
streams for animations.

Example:

```js
import tween from 'xstream/extra/tween'

const stream = tween({
  from: 20,
  to: 100,
  ease: tween.exponential.easeIn,
  duration: 1000, // milliseconds
})

stream.addListener({
  next: (x) => console.log(x),
  error: (err) => console.error(err),
  complete: () => console.log('concat completed'),
})
```

The stream would behave like the plot below:

```text
100                  #
|
|
|
|
80                  #
|
|
|
|                  #
60
|
|                 #
|
|                #
40
|               #
|              #
|            ##
|         ###
20########
+---------------------> time
```

Provide a configuration object with **from**, **to**, **duration**, **ease**,
**interval** (optional), and this factory function will return a stream of
numbers following that pattern. The first number emitted will be `from`, and
the last number will be `to`. The numbers in between follow the easing
function you specify in `ease`, and the stream emission will last in total
`duration` milliseconds.

The easing functions are attached to `tween` too, such as
`tween.linear.ease`, `tween.power2.easeIn`, `tween.exponential.easeOut`, etc.
Here is a list of all the available easing options:

- `tween.linear` with ease
- `tween.power2` with easeIn, easeOut, easeInOut
- `tween.power3` with easeIn, easeOut, easeInOut
- `tween.power4` with easeIn, easeOut, easeInOut
- `tween.exponential` with easeIn, easeOut, easeInOut
- `tween.back` with easeIn, easeOut, easeInOut
- `tween.bounce` with easeIn, easeOut, easeInOut
- `tween.circular` with easeIn, easeOut, easeInOut
- `tween.elastic` with easeIn, easeOut, easeInOut
- `tween.sine` with easeIn, easeOut, easeInOut

#### Arguments:

- `config: TweenConfig` An object with properties `from: number`, `to: number`, `duration: number`, `ease: function` (optional, defaults to
linear), `interval: number` (optional, defaults to 15).

#### Returns:  Stream 

- - -

