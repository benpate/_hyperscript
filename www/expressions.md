---
layout: layout.njk
title: ///_hyperscript
---

<div class="row">
<div class="2 col nav">

**Contents**

<div id="contents">

* [Introduction](#introduction)
* [CSS Literals](#css)
* [Comparisons](#css)
* [Strings](#strings)
* [Null Safety](#null-safety)
* [Array Expansion](#array-expansion)
* [Blocks](#blocks)
* [Async Expressions](#async)
* [Time Expressions](#time)

</div>

</div>
<div class="10 col">


## <a name="introduction"></a>[Introduction](#introduction)

Expressions in hyperscript are a mix of familiar and new.  Many of the expected operations from javascript are there:

* Common literals: `1 true "this is a string" null`
* List and object literals: `[1, 2, 3], {foo:"bar"}`

Most of the common comparison and mathematical operators are there as well:

```text
  x > 10
  y == null
  lst.length < 5
  a + b < 42
```

However, once you get past the basics, hyperscript starts to get a little wild.

Let's start with CSS literals.

## <a name="css"></a>[CSS Literals](#css)

Hyperscript gives you the ability to embed CSS literals directly in your code to select elements.  There are four
main expression types:

### ID Literals

You can refer to an element by ID directly in hyperscript as follows:

```html
<div _="on click put 'Clicked!' into #example.innerHTML">Click Me</div>
<div id="example"></div>
```

The `#example` is an ID literal and will evaluate to the element with the given id.  Here we put some text into its
`innerHTML` when the top div is clicked.

### Class Literals

You can refer to a group of elements by class directly in hyperscript as follows:

```html
<div _="on click put 'Clicked!' into .example.innerHTML">Click Me</div>
<div class="example"></div>
<div class="example"></div>
```

The `#example` is an ID literal and will evaluate all the elements with the class `example` on them.  Here we put some 
text into their `innerHTML` when the top div is clicked.  Note that the [put command](/commands/put) can work with
collections as well as single values, so it can put the given value into all the returned elements.

### Query Literals

Finally can refer to a group of elements by an arbitrary [CSS selector](https://www.w3schools.com/cssref/css_selectors.asp)
 by enclosing the selector in a `<` and `/>`:

```html
<div _="on click put 'Clicked!' into <div/>.innerHTML">Click Me</div>
<div class="example"></div>
<div class="example"></div>
```

This example will put "Clicked" into every div on the page!

```html
<div _="on click put 'Clicked!' into <div:not(.example)/>.innerHTML">Click Me</div>
<div class="example"></div>
<div class="example"></div>
```

This example will put "Clicked" into every div that does not have the `example` class on it.

### In Expressions

The `in` expression isn't a literal, but can be used in conjunction with them for common patterns:

```html
<div _="on click put 'Clicked!' into (<p/> in me).innerHTML">Click Me
  <p></p>
  <p></p>
</div>
```

The `in` expression in this case evaluates the query in the given context on the right hand side.  Here we are looking
up paragraph tags inside the clicked element (`me`) and setting their `innerHTML` to "Clicked!"

## <a name="comparisons"></a>[Comparisons](#comparisons)

In addition to the typical comparison operators, such as `==` and `!=`, hyperscript supports the following natural
language aliases

* `is` - equivalent to `==`: `1 is 1`
* `is not` - equivalent to `!=`: `1 is not 1`
* `am` - equivalent to `==`: `I am 1`
* `am not` - equivalent to `!=`: `I am not 1`
* `no` - equivalent to `!= null`: `no .example in me`

Note that `I` is an alias for `me`, the current element.

Furthermore, hyperscript supports two more comparison operators: `matches` and `contains` which can be used in the 
following forms:

```text
  I match <:hover/>
  it matches <:hover/>
  I do not match <:hover/>
  it does not match <:hover/>
```

```text
  I contain <:focus/>
  it contains <:focus/>
  I do not contain <:focus/>
  it does not contain <:focus/>
```

## <a name="strings"></a>[Strings](#strings)

Strings are similar to javascript, and can start with `"` or `'`.  The big difference is that you can use `$` for 
expression interpolation in regular strings.  Hyperscript does not support the backtic template syntax from javascript
because interpolation works with normal strings.

```html
<div _="on click set world to 'hyperscript' put 'Hello $world' into my.innerHTML">
  Click Me
</div>
```

In a few places, Hyperscript allowes "naked" strings, strings without a leading quote or double quote.  An example is
the [fetch command](/commands/fetch), which can take a URL as a naked string:

```html
<button _="on click fetch /example then put it into my.innerHTML">
    Fetch It!
</button>
```

Here the `/example` element is an example of a naked string.  Naked strings are ended by whitespace.

## <a name="conversions"></a>[Conversions](#conversions)

Hyperscript centralizes conversions into a single construct, the `as` expression:

```text
  10 as String
  "10" as Int
  "10.3" as Float
```

Out of the box hyperscript provides the following conversions:

* `String` - converts to string
* `Int` - converts to an integer
* `Float` - converts to a float
* `Number` - converts to a number
* `Date` - converts to a date

## <a name="null-safety"></a>[Null Safety](#null-safety)

The `.` operator in hyperscript is null safe, so `elt.parent` will evaluate to `null` if `elt` is null

## <a name="array-expansion"></a>[Array Expansion](#array-expansion)

Properties on arrays, except for `length`, are expanded via a [flat map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap)

```text
  set divs to </div>                      -- get all divs in the document
  set divParents to divs.parentElement    -- get all parents of those divs
  set divChildren to divs.children        -- get all children of those divs
```

## <a name="blocks"></a>[Blocks](#blocks)

Hyperscript does not have anonymous functions or complex arrow functions.  Because hyperscript is [async transparent](/docs#async)
complicated callbacks are generally not necessary.

However it does support a simple, expression-only version of arrow functions called "blocks", with a slightly different 
syntax:

```text
    set strs to ["a", "ab", "abc"]
    set lens to strs.map( \ s -> s.length )
```

Blocks start with a backslash, followed by args, then an `->` and then an expression value.

## <a name="async"></a>[Async](#async)

By default, hyperscript synchronizes on any [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 
that go through its runtime.  Consider the following code:

```html
<script type="text/hyperscript">
def waitThenReturn10()
    wait 10ms
    return 10
end
</script>
<button _="on click put 'The answer was $waitThenReturn10()' into my.innerHTML">
  Click Me...
</button>
```

Here we have an asynchronous function with a `wait` in it that will cause the function to return a Promise rather than
the value.  Hyperscript will "pause" evaluation of the event handler until that promise resolves and can provide a 
value to the string expression, and then continue.  This is very cool and is usually what you want.

However, there may be a case where you don't want hyperscript to pause and, instead, want to pass the raw promise on 
somewhere else.  To do this, you can use the `async` prefix for the expression

```html
<button _="on click call handleAPromise(async waitThenReturn10())">
  Click Me...
</button>
```

Here we pass the promise returned by `waitThenReturn10()` out to `handleAPromise()`, *as* a Promise, rather than resolving
it.

## <a name="time"></a>[Time Expressions](#time)

In a few places in the hyperscript grammar you can use "time expressions", which are just natural ways to specify
a time interval.  The [wait command](/commands/wait) is one such place:

```html
<button _="on click wait 2s then put 'I waited!' into my.innerHTML">
  Click to Wait...
</button>
```

You can use the following formats for time expressions:

* `10 ms` - milliseconds
* `10 milliseconds` - milliseconds
* `10 s` - seconds
* `10 seconds` - seconds

Note that a space between the number and modifier are not necessary, and that the `ms` modifier is just for clarity
since milliseconds is the normal interpretation for things like `setTimeout()`, which the wait command is based on.

</div>