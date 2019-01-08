# Parjs - Parser Combinator Library
[![build](https://travis-ci.org/GregRos/parjs.svg?branch=master)](https://travis-ci.org/GregRos/parjs)
[![codecov](https://codecov.io/gh/GregRos/parjs/branch/master/graph/badge.svg)](https://codecov.io/gh/GregRos/parjs)
[![npm](https://badge.fury.io/js/parjs.svg )](https://www.npmjs.com/package/parjs)

[API Documentation](https://gregros.github.io/parjs/)

Parjs is a JavaScript library of parser combinators, similar in principle and in design to the likes of [Parsec](https://wiki.haskell.org/Parsec) and in particular its F# adaptation [FParsec](http://www.quanttec.com/fparsec/).

It's also similar to the [parsimmon](https://github.com/jneen/parsimmon) library, but intends to be superior to it. Currently it has many more features, including:

1. Many more combinators and basic parsers.
2. Opt-in support for parsing complex Unicode
3. Written in ES6
4. Systematically documented
5. Advanced debugging features and ability to parse very complex languages.

Parjs is written in TypeScript, using features of ES6+ such as classes, getter/setters, and other things. It's designed to be used from TypeScript too, but that's not necessary.

Parjs is can be used on the client or the server. It's a pretty big library but when minified and gzipped it has a relatively small footprint.

| Version/Format  | Bundle         | Minified | Minified+Gzipped |
|-----------------|----------------|----------|------------------|
| Default         | < 180kb        | < 100kb  | < 25kb           |
| /w Unicode Data | < 350kb        | < 180 kb | < 50kb           |

To start using it, install the module using NPM and then use:

```ts
import {Parjs} from 'parjs';
```

Or in some cases:

```ts
var Parjs = require('parjs').Parjs;
```

Which imports the main module.
## Example Parsers
You can see implementations of example parsers in the `examples` folder:

1. [Tuple Parser](https://github.com/GregRos/parjs/blob/master/src/examples/tuple.ts) 
2. [String Format Parser](https://github.com/GregRos/parjs/blob/master/src/examples/string.format.ts) 
3. [JSON parser](https://github.com/GregRos/parjs/blob/master/examples/src/json.ts)
4. [Math Expression Parser](https://github.com/GregRos/parjs/blob/master/src/examples/math.ts)

## What's a parser-combinator library?
It's a library for building complex parsers out of smaller, simpler ones. It also provides a set of those simpler building block parsers.

For example, if you have a parser `digit` for parsing decimal digits, you can parse a number by applying `digit` multiple times until it fails, and then producing the consumed text as a result. Then you can use another *combinator*  to convert the result to a number.

By combining different parsers in different ways, you can construct parsers for arbitrary expressions and languages.

Here is how you might construct a parser for text in the form `(a, b, c, ...)` where `a, b, c` are floating point numbers. One feature of the expression is that arbitrary amounts of whitespace are allowed in between the numbers.

```ts
//Built-in building block parser for floating point numbers.
let tupleElement = Parjs.float();

//Allow whitespace around elements:
let paddedElement = tupleElement.between(Parjs.whitespaces);

//Multiple instances of {paddedElement}, separated by a comma:
let separated = paddedElement.manySepBy(Parjs.string(","));

//Surround everything with parentheses:
let surrounded = separated.between(Parjs.string("("), Parjs.string(")"));

//prints [1, 2, 3]:
console.log(surrounded.parse("(1,  2 , 3 )"));
```

In the above example, things like `Parjs.float()`, `Parjs.spaces`, and `Parjs.string(str)` are building-block parsers and things like `.between(p)` and `.manySepBy(p)` and *combinators* that work on existing parsers to give you new ones.

Parser-combinators can also emit informative error messages when parsing fails.

## What can you use it for?
Parsing, generally. You can parse all sorts of things:

1. A custom DSL specifying an algorithm for chicken counting.
2. Your own flavor of markdown, just to make things even more confusing.
3. A custom data-interchange format inspired by chess notation.

The possibilities are limitless.

Since it's written in JavaScript, it can be used in web environments.

## Unicode and Limitations
Parjs supports selectively parsing diverse Unicode characters with the aid of the [`char-info` ](https://www.npmjs.com/package/char-info) package of character recognizers.

Parsers such as `Parjs.upper` can only recognize the ASCII subset of Unicode. Parsers beginning with `uni`, such as `uniUpper`, can recognize any Unicode character. However, they are also much slower as each character must be looked up in a tree-like data structure.

`Parjs` does not import `char-info` by default due to the latter's large size. In order to enable unicode parsing support, including calling functions with names beginning with `uni`, you need to import `parjs/unicode` somewhere in your code.

```ts
import 'parjs/unicode';
```

Doing so will load all the unicode data.

In order to parse Unicode characters with elaborate properties, you should install the `char-info` library and use its characters indicators directly. It lets you detect a character's Unicode category, Block, Script, and other information.

Parjs isn't very good at parsing characters outside of the BMP (Basic Multilingual Plane). In particular, even parsers beginning with `uni` won't recognize such characters. One reason for this is because JavaScript has a UCS-2 conception of characters.

## Module Structure
Parjs has a well-organized module structure that is reflected in the documentation:

    - parjs
        Contains all objects and TypeScript type 
        declarations needed to use the library for parsing.
           
        - parjs/internal
            Contains additional objects and type declerations that may be needed 
            to extend the library.
          
            - parjs/internal/implementation
                Contains objects and declerations for implementing additional parsers.
                
                - parjs/internal/implementation/combinators
                    Implementations of combinators.
                    
                - parjs/internal/implementation/parsers
                    Implementations of building-block parsers.
                    
                - parjs/internal/implementation/functions
                    Character recognizers and other functions used with parsing.


## What's a Parjs parser?
In `Parjs`, a parser is an object that consumes characters from text and returns a value. The number of characters the parser consumes depends on its implementation.

When a parser is invoked on a top level, it is expected to consume the entire input. If it does not, this signals an overall parsing failure. During the parsing process, a `position` value is maintained.

When a parser is invoked as part of a containing parser (e.g. `Parjs.seq(p1, p2)`), then the containing parser chooses how to handle the failure, using information such as the kind of failure and where it occurred. It also chooses how to handle the return value.

When several parsers are strung together in sequence inside a containing parser, the containing parser generally chooses how to apply those parsers. Typically, combinators such as `p1.then(p2)` apply the first parser until it consumes all the input it wants, and then apply the 2nd parser at the exact position the previous parser stopped consuming.

The result of executing a parser is called a *Reply*. 

## Immutability
It's important to note that parsers are meant to be immutable objects, and the library is designed around that important premise. 

More specifically, instances of parsers should not depend on instance-level information to process data. You can still edit the parser prototypes, adding combinators and building block parsers.

This allows you to write such idiomatic code as:

```ts
let myString = Parjs.string("my personal string");
let variant1 = myString.then(Parjs.string(" is the best."));
let variant2 = myString.then(Parjs.string(" is okay."));
```

If `myString` were a mutable object, mutating it in one part of the program would then change how `variant1` and `variant2` behave, which would be very suspicous behavior.

In practice, you can design parsers that don't behave this way, but doing so is highly discouraged.

## Parsing Failure
Parsers can fail, and this is completely normal in many situations. The `p.or` and `Parjs.any` combinators assume that some of the parsers will fail and will attempt to apply other parsers if that happens. They are failure recovery combinators.

Some failures though indicate *unexpected input* and shouldn't be swallowed by those constructs. For example, consider a parser that parses a floating point number with an optional exponent, such as `1.0e+10`, followed by an arbitrary string. If we're given the input `1.0e+` we *don't* want to parse it as `1.0` followed by the string `e+`. That would obscure what's likely an error in the input.

When an internal parser fails, the containing parser will generally propagate the same or similar parser. When there is no containing parser, a failure result will be emitted, indicating that the overall parsing has failed. In some cases, a soft failure in a child parser indicates a hard failure in a parent parser.

### Soft failures
Most simple failures are soft failures. They happen when a parser rejects the input immediately. For example, the parser `Parjs.digit` rejects the input `a` immediately and so fails softly. 

Recovering from soft failures doesn't require backtracking.

You use the `or` failure recovery combinator to handle soft failures, generally by offering several alternative parsers to parse the text.

#### Hard failures
Hard failures happen when a parser fails after consuming some input. For example, the compound parser `P = a.then(b)` will fail hard if `a` succeeds but `b` fails. The rationale behind this is twofold. 

Firstly, `a` consumes some input before `b` fails, which means that the parser has developed an *expectation* that after `a` succeeds, `b` should also succeeds. When this expectation breaks, an error arises.

Secondly, recovering from `P` requires backtracking to the starting position of the parsing. Parjs is written to backtrack as little as possible, and so a failure that requires backtracking is more severe than one that does not.

For example, `p.then(q)` will fail hard if `q` fails softly, because this parser first applies `p` that consumes some of the input, and only then applies `q`. 

Another example is `p.exactly(3)`. This parser applies `p` exactly 3 times. If it fails to apply `p` even once, it will fail softly. But if `p` fails on the 2nd application, it will fail hard.

Similarly, the `.must` combinator can cause a parser to fail hard. If you apply`p.must(condition)` and `p` succeeds but `condition` fails, then the parser will fail hard because `p` already consumed some of the input. 

The `.soft` combinator translates hard failures into soft ones.

#### Fatal failures
This type of failure is raised on purpose to explicitly signal malformed input. It can be intentionally signaled to catch certain kinds of syntax errors and treat them accordingly. It cannot be recovered from using standard combinators. Even the `.not` combinator, which normally succeeds if the input parser fails, still propagates a fatal failure.

#### Exceptions
Exceptions aren't really part of this hierarchy. Parsers do not and should not throw exceptions to indicate invalid input, and Parjs does not handle thrown exceptions. Rather, an exception indicates a problem with the parser itself.

### Overall Parsing Failure
An overall parsing happens when a parser is invoked by you (the user), and either fails in any manner or fails to consume the entire input (which translates to a failure).

The result from a parsing operation that has failed is of the `FailureResult` type and exposes several important proprerties:

1. The `kind` of the failure.
2. The `trace` object which contains tracing information indicating where the parser failed, and what input was expected. It also contains the parser `userState` at the time of the failure.

In addition to emitting a failure result, parsers can also throw exceptions, as mentioned previously. This indicates an error in the parser.

## Quiet Parsers
Earlier I made the claim that all parsers return values. That's not exactly true. There are actually two kinds of parsers: loud and quiet parsers. Whether a parser is loud or quiet is an intrinsic property that is reflected in the TypeScript type system. It's not something that changes based on the input.

In principle, quiet parsers don't return values, only whether parsing succeeded or failed (they may also modify the user state, see more on that below). In actuality, they do return a special signalling value, but that value is ignored.

Quiet parsers are treated differently by combinators. For example, the `Parjs.seq(p1, p2)` combinator can accept both loud and quiet parsers. It applies parsers in sequence and returns an array of their results. Since quiet parsers aren't considered to return values, they aren't included. Thus `Parjs.seq(loud1, quiet, loud2)` will always return an array with 2 elements. 

Combinators that use the return value of a parser also behave differently, since there is no value to be projected.

Quiet parsers are an important feature of `Parjs`. There are many situations in which you don't care about the return value of a parser and what it to be ignored in aggregation combinators such as sequential ones. 

For this reason, the combinator that turns any parser into a quiet parser is called simply `.q`. 

```ts
let comma = Parjs.string(",").q;
let hello = Parjs.string("hello").q;
```

It's not an error to quieten an already quiet parser, but doing so does nothing and may return the exact same parser instance.

```ts
let comma = Parjs.string(".").q.q.q.q.q;
```

## User State
User state is a powerful feature that can be used when parsing complex languages, such as mathematical expressions with operator precedence and languages like XML where you need to match up an end tag to a start tag.

Basically, when you invoke the `.parse(str)` method, a unique, mutable user state object is created that is propagated throughout the parsing process. Every parser can read and edit the current parser user state. Built-in parsers aren't allowed to use the user state directly (they can do other things), so the only information in it will be what you put inside it.

The `.parse` method accepts an additional parameter `initialState` that contains properties and methods that are merged with the user state:

```ts
//p is called with a parser state initialized with properties and methods.
let example = p.parse("hello", {token: "hi", method() {return 1;});
```

Here is an example of how you can use this feature to parse a recursive, XML-like language:

```ts
// Define our identifier.
// Starts with a letter, followed by a letter or digit.
// The .str operator stringifies the array of characters.
let ident = Parjs.letter.then(Parjs.digit.or(Parjs.letter).many()).str;
//A parser that parses an opening of a tag.
let openTag = ident.between("<", ">").each((result, {tags}) => {
    tags.push({
        tag: result,
        content: []
    });
}).q;

// The close tag is </ TAG >.
let closeTag =
    ident.between("</", ">").each((result, {tags}) => {
        let topTag = tags.pop();
        tags[tags.length - 1].content.push(topTag);
    }).q;

let anyTag =
    closeTag.or(openTag).many().state.map(x => x.tags[0].content)
        .isolateState({tags: [{content: []}]});
let parsedXmlData = anyTag.parse("<a><b><c></c></b></a>");
```

Among other uses, user state allows you to parse operator precedence using LR parsing techniques even though Parjs is essentially a library for LL parsers.

Many methods that project the result of a parser take a function with two arguments, the first being the result and the 2nd being the state object. Quiet parsers support projection methods that operate exclusively on the state.

User state is a less idiomatic and elegant feature meant to be used together with, rather than instead of, parser returns.

You can also make use of the advanced     `isolateState` combinator. This combinator lets you isolate a parser's user state from other parsers. This lets you write a black-box parser that still uses user state. In the above example, it's used to make sure the user state has the correct structure when the `anyTag` parser is entered.

## Implicit parser literals

When a combinator expects a `LoudParser<string>` as input, you can actually substitute a string. The string `str` will be implicitly understood to be `Parjs.string(str)`, i.e. a parser that parses that string and returns it.

Strings are an example of implicit parser literals. Another example is a regular expression, which is implicitly converted using `Parjs.regexp`. 

This fully type checks using TypeScript.

## Parsers and combinators
This is a partial overview of the kinds of parsers and combinators provided by `Parjs`. This is not an exhaustive list.

### Basic Parsers
These are building block parsers provided by `parjs`. These are generally defined in [ParjsStatic](https://gregros.github.io/parjs/interfaces/parjs.parjsstatic.html), i.e. the type as the object `Parjs`.

#### Character parsers
One of the most common kinds of parser, parses individual characters.

1. `Parjs.anyChar` - Parses any single character.
2. `Parjs.anyCharOf(str)` - Parses any single character that appears in `str`.
3. `Parjs.digit` - Parses a single digit.
4. `Parjs.asciiLetter` - Parses a single ASCII letter character.

#### String parsers
Parses entire strings.

1. `Parjs.string(str)` - Parses the exact string `str` or fails softly.
2. `Parjs.rest` - Parses the remaining text (if any) and returns it as a string.
3. `Parjs.regexp(rxp)` - Applies the regular expression `rxp` at the current position and returns an array of the match groups.

#### Numeric parsers
These parse multiple characters as numbers, either integers or floating point.

1. `Parjs.int(?options)` - Parses an integer using `options`. If no options object was specified, default options are used.
2. `Parjs.float(?options)` - Parses a floating point number using `options`. These differ from integer parsing options. If no options object was specifie, default options are used.

#### Primitive parsers
These parsers are very simple and don't consume any input.

1. `Parjs.nop` - A quiet parser that consumes no input and returns no value.
3. `Parjs.result(v)` - A loud parser that consumes no input and returns `v`.
4. `Parjs.fail(args)` - A loud parser that always fails with failure information specified in `args`.

#### Special parsers
These special parsers don't belong to any group.

1. `Parjs.position` - A parser that succeeds without consuming input and returns the current position in the stream.
2. `Parjs.state` - A parser that succeeds without consuming input and returns the current parser state.

### Combinators
 These are defined in [`ParjsStatic`](https://gregros.github.io/parjs/interfaces/parjs.parjsstatic.html) statically, [`LoudParser`](https://gregros.github.io/parjs/interfaces/parjs.loudparser.html) for loud parser instances, and [`QuietParser`](https://gregros.github.io/parjs/interfaces/parjs.quietparser.html) for quiet parsers.

Here `P` refers to the parser created by the combinator.

#### Projections
These combinators create parsers that project the result of the input to a different form. In

1. `p.map(f)` P applies the function `f` to the result returned by `p`.
2. `p.str` P stringifies the result returned by `p`. This means something different for different types. For example, arrays of strings are flattened and concatenated. 
3. `p.each(f)` P applies the function `f` to the result returned by `p` and returns the same thing.
4. `p.q` - P applies `p` and returns nothing. A quiet parser.
5. `p.state` - P applies `p`, ignores its return value and instead returns the parser state object.

#### Assertions
These combinators check if a condition applies, and fail if it does not. They accept additional arguments that specify the kind of failure. They don't change the result.

1. `p.must(f)` - P applies `f` on the result of `p` and fails if it retursn false.
2. `p.mustBeNonEmpty()` - P fails if the result of `p` is "empty". This includes various values and is not the same as falsy.
3. `p.mustCapture()` - P fails if `p` succeeds without consuming input.

#### Sequential
These combinators apply a number of parsers sequentially.

1. `p1.then(p2)` - P applies `p1` and then `p2`. The result depends on the loudness of `p1, p2`. A highly overloaded combinator.
2. `p2.between(p1, p3)` - P is identical to `p1.then(p2).then(p3)`, except that it returns only the value of `p2`.
3. `p.many(args)` - P applies `p` until it fails softly. Accepts arguments that indicate minimum number of successes required and other information.
4. `p.manySepBy(sep)` - P applies `p` multiple times, every two occurrences separated by `sep`.
5. `Parjs.seq(p1, p2, p3)` - Applies the parsers `p1, p2, p3` in sequence. Returns an array of the results. Quiet parsers don't contribute to the array.
6. `p.thenChoose(selector)` - P applies `p` and then calls `selector` on the result, which returns the parser to apply next.

#### Alternatives
These combinators try several parsers in sequence until one of them succeeds. They are a subtype of failure recovery combinators.

1. `p1.or(p2)` - P applies `p1`. If `p1` fails softly, applies `p2` at the same position. Highly overloaded combinator. You cannot mix loudess with this combinator -- e.g. `loud.or(quiet)` is a runtime error (and a compilation error in TypeScript).
2. `p1.orVal(v)` - P applies `p1`. If `p1` fails softly, succeeds and returns `v` without consuming input.

#### Primitive
These combinators are very simple.
2. `p.fail(args)` - P applies `p` and fails with `args` if it succeeds. Also propagates failures.

#### Special

1. `p.not` - P succeeds without consuming input or returning a value if `p` fails hard or soft at the current position. If `p` succeeds, P fails softly. Propagates a fatal failure. A quiet parser.
2. `p.backtrack` - P applies `p`, backtracks to the original position in the input (before applying `p`), and returns the result. 

## The reply of a parser
When a parser `p` is applied using the `p.parse(str)` method, a `ParserReply<T>` is returned. This reply is either a success or a failure of a differing severity.

Every `reply` has a `reply.kind` value, which is a string that can be any value that is part of the `ReplyKind` set: `OK, Soft, Hard, Fatal`. To check that parsing succeeded, just compare `kind` to one of these values:

```ts
let reply = p.parse(str);
if (reply.kind === "OK") {
    //parsing succeeded
}
else {
    //parsing failed
}
```

When using TypeScript, the check narrows the type of `reply` in each respective branch.

The `ParserReply<T>` exposes a `value` property that returns the result of the parser, if it succeeded. But failure causes an exception to be thrown.

### Success Reply
This is the reply you usually hope to get when parsing something. It indicates that parsing succeeded.

The primary property is `value`, which exposes the reply value of the parser. Note that the user state at which the parser finished is swallowed and isn't returned (see more about user state in another part of the documentation).

### Failure Reply
A failure reply is when `kind !== "OK"`. The `kind` could be any of the failure kinds: Soft, Hard, or Fatal failure.

In addition to the `kind`, a failure reply exposes the `trace` property that describes the circumstances of the failure in a systematic way.

1. `userState`, which exposes the last user state the parser failed with. Note that using advanced combinators like `isolate` hides the entirety of this information.
2. `position`, which exposes the position (in terms of JavaScript characters) at which the parser failed.
3. `reason`, which exposes the reason for why the parser failed, such as `"expected ','"`
4. `location`, exposes the row and column at which parsing failed. This is deduced using a simple algorithm that assumes a monospaces font and no combining diacritics and may be incorrect for especially complex texts.
5. `stackTrace`, a parser stack trace.
6. `input`, contains the input text.

A visualizer is provided that can graphically display the error location. TO access the visualizer, do:

```ts
Parjs.visualizer.visualize(reply.trace);
```

The result is a textual representation of the error.

## Implementation
Parjs keeps interface and implementation totally separate.

Parsers are primarily created from scratch using the `Parjs` object, of type `ParjsStatic`. They can come in two interfaces: `LoudParser<T>` and `QuietParser`. Each of these inherits from the interface `AnyParser` that has the chraracteristics and combinators supported by both.

Internally, the `Parjs` object actually is of the class [`ParjsParsers`](https://gregros.github.io/parjs/classes/parjs_internal.parjsparsers.html).

All parsers, whether quiet or loud, are instances of the class [`ParjsParser`](https://gregros.github.io/parjs/classes/parjs_internal.parjsparser.html) wrapping an internal object of the class [`ParjsAction`](https://gregros.github.io/parjs/classes/parjs_internal_implementation.parjsaction.html). 

Loudness or quietness is communicated via the `isLoud` property of the action and the wrapping parser -- otherwise loud parsers and quiet parsers are identical. However, functions will throw exceptions if the loudness of the input parser was not as expected.

## Writing a parser with custom logic
**In most cases, it should be easy to use existing combinators and building block parsers to create custom parsers. **

However, `Parjs` is meant to be very easy to extend, so if you can't use existing code to do your work for you, writing your own is very simple

### Parser flow
When the `.parse` method is called, a [`ParsingState`](https://gregros.github.io/parjs/interfaces/parjs_internal.parsingstate.html) object is created. This is a mutable object that indicates the state of the parsing process. Here are some of its members:

```ts
interface ParsingState {
    readonly input : string;
    position : number;
    value : any;
    userState : any;
    expecting : string;
    kind : ReplyKind;
    //...
}
```

Every parser is a wrapper around a thinner object called a *parser action*. The chief method if this action is the `apply(ParsingState)` method that mutates the parsing state. By doing so, it can return a value, advance the position, mutate the user state, and signal failure or success.

The `apply` method of a parser action involves some boilerplate, so you don't have to implement it directly. Instead, you have your parser action extend `ParjsAction`. This is an abstract class that implements the `apply` method and delegates most of its work to an internal `_apply(ps)` method. 

There are several important rules to writing a parser action:

1. The action is required to set the `kind` field of the `ParsingState`. This is how the action communicates success or failure. 
2. If the action returns a value (e.g. `isLoud` returns `true`), it must also set the `value` member as this is how it communicates the return value.

Failure to do either of these will cause an exception to be thrown.

Here is an example of one implementation. This implementation checks if parsing has reached the end of the input (e.g. `Parjs.eof`).

```ts
_apply(ps : ParsingState) {
    if (ps.position === ps.input.length) {
        ps.kind =  ReplyKind.OK;
    } else {
        ps.kind = ReplyKind.SoftFail;
    }
}
```

This specific action is quiet, so it doesn't need to set the `value` property. 
​    
### Other required properties of parser actions
In addition to implementing `_apply`, parser actions must also specify:

1. The `isLoud` property, which says whether the action returns a value. It's important that this property not change after the parser is returned to the user, because loudness is reflected in the TypeScript type system via different interfaces.
2. The `expecting` property, which is a text specifying what input the parser action is expecting to parse. For example, it could be `a digit`, `end of input`, or something else. The text is generally set when the parser is constructed. It is needed for displaying relevant debugging information.

### Creating a parser
After you have written your parser action, you'll need to wrap it in a parser. The default parser class is `ParjsParser`, which contains implementations for all the relevant combinators as prototype members.

```ts
 let parser = new ParjsParser(action);
```
Before returning it, also call its `withName` method to set the parser's display name.

Finally, if you are writing in TypeScript, you'll need to cast the parser to the appropriate interface. This is usually either `LoudParser<T>` or `QuietParser`.    
