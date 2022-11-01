# 32chars.js

## Disclaimer

The program in this repository has the potential to be used maliciously, such as injecting obfuscated malicious code in websites. **I am solely never held responsible for any damage caused by this program or the code it outputs.**

## Introduction

32chars.js is a JavaScript encoder that obfuscates a piece of text into the shortest possible valid JavaScript code with only 32 ASCII symbols and punctuation: "`~!@#$%^&\*()\_+{}|: "<>?-=[]\;',./". This project is a testament to the many existing JavaScript encoders and the spiritual successor to the original encoder, [jjencode](https://utf-8.jp/public/jjencode.html).

## Background

We all know JavaScript is a programming language with many weird and tricky parts. Because of this, one can write JavaScript without the need for letters or numbers, perhaps only with [five](https://aem1k.com/five/) or [six](http://www.jsfuck.com/) possible symbols. However, all these would result in _extremely verbose_ code, where a single character might expand to _thousands_ of characters. The only practical use case for this is for people to obfuscate their code.

Meanwhile, most program code contains various letters, numbers, symbols, and spaces, making code shorter, more succinct, and readable. This project aims to generate obfuscated code "just right," with a character set and output length "just right." That means no letters, numbers, or spaces but all 32 ASCII punctuation and symbol characters, most of which are significant JavaScript tokens. Therefore, this project aims to produce the minimum possible JavaScript encoding using those 32 characters.

So rather than going over each character in the input string one by one and then expanding it, we are breaking the input string into runs with a length of at least one. This way, we are effectively minimizing the number of operations performed by the JavaScript engine while shortening the output program's length. The compiler encodes the string piecewise, so parts of the input string remain untouched. By working backward, this compiler determines the set of JavaScript's built-in operations to produce the original string.

## Explanation

We have a large enough character set, and most of these symbols also are valid JavaScript tokens. The most significant of them are:

- `'`, `"` and `` ` `` to delimit strings. The backtick "` "delimits template strings, which allow for interpolation and tagged function calls
- Combinations of `_` and `$` to represent variables and identifiers
- `+` to concatenate strings, add numbers and cast things into numbers
- Arithmetic `+ - * / % **` and bitwise `& | ^ ~ << >> >>>` operators with a `Number` and `BigInt` on either side.
- `.` to access properties of objects
- `,` to delimit array, object, and function elements
- `()` to call functions and group expressions
- `[]` to access array and object elements and create array literals
- `{}` to delimit objects and blocks of code
- `${}` to interpolate expressions in template strings
- `/` to delimit regular expression literals

Strings are fundamental to obfuscation, as we can encode and store custom data without breaking the compiler. However, there are other ways to create strings with some of JavaScript's newest syntax and functionality, such as `RegExp`, `BigInt`, and JavaScript's new `String`, `Array`, and `Object` methods.

The program uses a two-phase substitution encoding. First, characters and values are assigned to variables and properties, decoded, and used to construct new substrings or build the input string. There are, of course, edge cases, as some expressions innately produce strings, such as constants (like `true`, `false`, `0`), string tags (like `[object Object]`), date strings (like `2011-10-05T14:48:00.000Z`) and source code (like `function Array() { [native code] }`).

The text is parsed and assigned to tokens: symbol sequences using those 32 characters, integers, ASCII characters, words with diacritics, non-Latin writing systems, Unicode BMP (Basic Multilingual Plane) characters (code point `U+0000` to `U+FFFF`), and astral characters (code point `U+10000` and above). JavaScript stores strings as UTF-16, so all BMP characters would receive _two_ bytes while the rest would receive _four_.

The text is split by default with the space. The split elements go into an array, delimited with commas `,`. The split result can also have an empty string, so nothing goes in between the commas. Unfortunately, JavaScript ignores trailing commas in arrays, objects, and function arguments, so we have to explicitly add trailing commas if the final element of an output array is empty.

### Representing strings

String literals in JavaScript can be created in three ways: single and double-quoted strings, which are roughly equivalent, and template strings, which allow interpolation of values. In this context, strings are _very fundamental_ to obfuscation, so we are free to create our encoding/decoding schemes.

In the output string, substrings with only printable ASCII symbols are quoted into string literals. Usually, the runs do not contain any quote characters `` ' " ` ``. The backslash `\`, and whatever symbol is being used for wrapping these literals, are escaped. In template literals, the sequence `${`, which begins an interpolation sequence, is also escaped.

Each encoded substring in the output goes through a quoting function (from the `jsesc` library) which compares the lengths of the escaped substring and selects the string literal with the least number of escapes, in that case being the shortest. It also prioritizes a fallback quoting option for strings without escapes.

### Representing strings

One can create string literals in JavaScript in three ways: single and double-quoted strings, which are roughly equivalent, and template strings, which allow interpolating values. In this context, strings are _very fundamental_ to obfuscation, so we are free to create our encoding/decoding schemes.

In the output string, we quote substrings with only printable ASCII symbols are quoted into string literals. Usually, the runs do not contain any quote characters `` ' " ` ``. Instead, the backslash `\`, and whatever symbol is being used for wrapping these literals, are escaped. In template literals, the sequence `${`, which begins an interpolation sequence, is also escaped.

Each encoded substring in the output goes through a quoting function (from the `jsesc` library) which compares the lengths of the escaped substring and selects the string literal with the least number of escapes, in that case being the shortest. It also prioritizes a fallback quoting option for strings without escapes.

### Statement 1

JavaScript allows the `_` and `$` characters to be used in variable names. So we can assign one of them, in this case, `$`, to be used to store values, characters, and substrings, and the other, `_`, to store the actual string.

The code starts by assigning `$` to the value of `-1`, or by doing a bitwise NOT on an empty array: `~[]`. An empty array, or implicitly, a _string_, is `0`, and `~0` is equal to `-1`.

### Statement 2

In the following statement, we assign `$` is assigned to a JavaScript object. We define properties within the braces in the form `key: value`, and individual properties, key-value pairs, are separated with commas.

JavaScript has three different ways to represent keys: _identifiers_ without quotes (which also includes keywords) like `key:value`, _strings_ within single or double quotes like `key':value`, and _expressions_ within square brackets `['key']:value`. In addition, one can access object properties with dots, like `x.key`, or square brackets, like`x['key']`.

The first property is `___`, with a value of `` `${++$}` ``. This takes the value of `$`, currently `-1`, and increments it to `0`, then casts that into a string by wrapping it inside a template literal interpolation (implicitly calling `.toString`). While the object is still being built, `$` is still a number and not an object yet, since evaluation happens from the inside out.

The second property is `_$`, with a value of `` `${!''}`[$] ``. An empty string is prepended with the NOT operator, coercing it into `false` since it is "false"-y. `!` also negates the boolean, resulting in `true`, before wrapping it in a template literal, becoming the string `"true"`. The `[$]` construct returns the character at index `0` (string indices start at `0`), which returns `t`.

We construct the rest of the properties similarly: incrementing the `$` variable, constructing a string, and grabbing a character out of the string by specifying its index. Next, we manipulate literals to evaluate to form the constants, `true`, `false`, `Infinity`, `NaN`, `undefined`, and an empty object `{}` which becomes the string `" [object Object]" `.

The constants would yield the following letters (case-sensitive): a b c d e f i j I l n N o O r s t u y', the space and the digits `0 1 2 3 4 5 6 7 8 9`. We have syntax to form the hexadecimal alphabet and several uppercase and lowercase letters.

The space, the only non-alphanumeric and non-symbol character is assigned the property `-`.

We encode the numbers in binary, substituting `_` for digit `0` and `$` for digit `1`. Then we pad the result to a length of 3 by adding `_` (as `__`, `_$`, `$_` and `$$` have keys defined). So `___` is `0` and `__$` is `1`.

We encode the letters as a pair of characters. The first `_` or `$` defines its case, and the second a symbol, each unique to a letter in the English alphabet, minus the three pairs of brackets `()[]{}`. The most common letters in English, `t` and `e` get the characters which form identifiers, `_` and `$`, while the least common, `j`, `q`, `x` get the quote characters, while `z` gets the backslash.

### Statement 3 and 4

From the third line, we will use the letters we have formed to begin forming the names of properties in our expressions by concatenating them with the `+` operator. We form the words `concat`, `join`, `slice`, `return`, `constructor`, `filter`, `flat` and `source`, each assigning single or double character keys.

`[]` can be used to access properties on values and, by extension, to call methods on them. For instance, `[]['flat']()` is semantically equivalent to `[].flat()`.

We can now access the constructors with the `constructor` property of literals: `Array` `[]`, `String` `''`, `Number` `+[]`, `Boolean` `![]`, `RegExp` `/./` and `Function` `()=>{}` (leaving out `Object`), and like before, casting that constructor function into a string. When evaluated in a JavaScript interpreter, this yields a string like `function Array { [native code] }`. We now have the letters `A B E F g m p R S v x`. The only letter not present in any of the constructor names is `v`, which is from the word `native`.

Now we have 22 lowercase `a b c d e f g i j l m n o p r s t u v x y` and 9 uppercase letters `A B E F I N O R S`. Furthermore, like before, we store every word and letter we have formed with a unique key for reference later in our code.

We use the spread operator, `...`, to "spread out" its properties on a new object, in this case, itself, before reassigning it to itself.

In statement 4, by concatenating the letters, we form the method names `map`, `replace`, `repeat`, `split`, `indexOf`, `entries`, `fromEntries`, and `reverse`.

### Statement 5 and beyond

We are going to get the following letters: `h k q w z C D U`, make the strings `toString`, `fromCharCode`, `keys`, `raw`, and `toUpperCase`, and retrieve the functions `Date, BigInt`, `eval`, `escape` and `parseInt`.

`toString` is formed by concatenating the letters `t` and `o', and then retrieving the string `"String"`from the`String`constructor, by accessing its`name`property. With `toString`, we can retrieve the rest of the lowercase alphabet,`h k q w z`, by passing a number in base 36, to yield a letter in lowercase.

Using the `Function` constructor, we can trigger the execution of code contained in a string as if it was native JavaScript code. For example, with an expression such as `Function('return eval')`, we can retrieve critical global functions, such as `eval`, `escape`, and `parseInt`.

The letters `C` and `D` are created by indexing a URL string with an invalid character (not an ASCII letter, digit or the characters `-`, `_`, `.`, and `~`), that is, `<` (`%3C`) or `=` (`%3D`) with the `escape` function and prefixes it with a percent `%`. The `escape` function yields these escape sequences in uppercase.

The expression `{}.toString.call().toString()` creates the letter `U`, which derives from the string `[object Undefined]`. With it, we can form the method `toUpperCase`, which does what it says: convert an entire string into uppercase. Using this method, we can get the remaining uppercase letters and then reassign them with array destructuring. This lets us assign and swap variables by putting their values in an array on both sides.

Both `eval` and `fromCharCode` allows us to form Unicode strings. `fromCharCode` generates a string from its code points. In contrast, `eval` generates a string from its escape sequence. `parseInt` enables numbers to be parsed in bases other than 10.

When used on a template literal, the `String.raw` method ignores all escape sequences, so backslashes are now interpreted as they are without getting "deleted" by the parser.

> This depends entirely on the current locale and JavaScript engine so this feature is considered _experimental_. The additional letters `G M T J W Z` can be retrieved with the `Date` constructor:
>
> - The letters `G M T` is formed from the expression `new Date().toString()`. This yields a string of the form `Thu Jan 01 1970 07:30:00 GMT+0XXX (Local Time)`.
> - `Z` comes from `new Date().toISOString()` which evaluates to a string of the form `1970-01-01T00:00:00.000Z`. `Z` in this case represents zero UTC offset.
> - Passing these arguments to the `Date` constructor, in a specific order, retrieves `J` and `W`: `Jan` - `0`, `Wed` - `0,0,3`.

```js
const props = {
  // statement 2: constants
  // 0 1 2 3 4 5 6 7 8 9
  // a b c d e f i j I l n N o O r s t u y [space]
  space: "-",

  // statement 3
  concat: "+",
  call: "!",
  join: "%",
  slice: "/",
  return: "_",
  constructor: "$",
  source: ",",

  // statement 4: constructors
  // A B E F g m p R S v x

  // statement 5
  name: "?",
  map: "^",
  replace: ":",
  repeat: "*",
  split: "|",
  indexOf: "#",
  entries: ";",
  fromEntries: "< ",
  reverse: '"',

  // statement 6: constructors
  // 'to' + String.constructor.name
  // C, D (from 'escape')
  toString: "  '",

  // statement 7: global functions
  eval: "=",
  escape: ">",
  parseInt: "~",

  // statement 8: toString, escape and call
  // h k q w z U

  // statement 9
  fromCharCode: "@",
  keys: "&",
  raw: " `",
  toUpperCase: ".",
}
```

### Encoding

The output ships with several functions, defined with numeric keys within the range `-1` to `-2`. These functions are stored inside numeric keys in the global object, minified with UglifyJS, and further processed using a custom algorithm which substitutes all the letters and numbers with sequences of letters. The script is then passed onto `eval` as an anonymous function, with the calls between the functions substituted inside the string.

One pair compresses and expands ranges of Unicode characters with reserved delimiters, while another encodes and decodes strings to and from `BigInt's, using a given character set. Because of the concept of [bijective numeration](https://en.wikipedia.org/wiki/Bijective_numeration), a generator function yields every possible string with those same 32 characters in order and skips the keys already defined.

If a substring within the input occurs more than once or appears just frequently enough based on Jaro distance, then it will be encoded and stored in the global object. All encoded strings are grouped according to their writing system, decoded by looping over the keys, and finally spread out into the global object.

<!-- prettier-ignore -->
```js
function encodeBijective(n,e){e=[...new Set(e)];var t=BigInt,r=t(e.length),i=e[((n=t(n))%r||r)-1n];if(!(0n<n))return"";for(;0n<(n=(n-1n)/r);)i=e[(n%r||r)-1n]+i;return i}
function decodeBijective(e,n){n=[...new Set(n)],e=[...e];for(var t=BigInt,i=0n,r=t(n.length),c=e.length,d=0;d<c;d++)i+=t(n.indexOf(e[d])+1)*r**t(c-d-1);return i}
function compressRange(e,n,t=",",o="."){return n=[...new Set(n)].filter(e=>e!=t&&e!=o).join``,[...new Set(e)].map(e=>e.codePointAt()).sort((e,n)=>e-n).reduce((e,n,t,o)=>{var r=o[t-1],i=n-r;return 0<t&&i==r-o[t-2]?(e[r=e.length-1][1]=n,1<i&&(e[r][2]=i)):e.push([n]),e},[]).map(e=>e.map(e=>encodeBijective(e,n)).join(o)).join(t)}
function expandRange(e,p,t=",",l="."){return p=[...new Set(p)].filter(e=>e!=t&&e!=l).join``,e.split(t).map(e=>{var t,a,n,r,i,o,e=e.split(l).map(e=>parseInt(decodeBijective(e,p)));return 1==e.length?e:([e,t,a,n]=[...e],r=(Math.abs(t-e)+2*(n||0))/(a||1)+1,i=e-(t=e<t?1:-1)*(n||0),o=t*(a||1),[...Array(r).keys()].map(e=>i+o*e))}).flat().map(e=>String.fromCodePoint(e)).sort((e,t)=>e.localeCompare(t)).join``}
```

#### Similar strings

Using the string methods `join`, `slice`, `replace`, `repeat`, and `split`, we can employ algorithms by working backward to help us produce these strings because the goal of this program is to minimize repetition.

Sometimes, the program does not encode specific substrings as we either have formed them and stored them in the global object or are magically created by manipulating primitive values, such as `function`, `Array`, `undefined`, `object`, and more.

- The `slice` returns consecutive characters within a substring by specifying a start and optional end index, for instance, `fin` and `fine` from `undefined`.
- The `repeat` method can repeat a "factored" substring many times, using a regular expression to determine the shortest pattern within the substring and how much it repeats.
- The `split` and `join` method is used to join an array of "strings" with a string as its delimiter. Then, using regular expressions, we determine the optimal substring to delimit that is not a space.
- The `replace` method is used to replace one or all substrings of a substring through insertion, deletion, or substitution to turn it into something similar. We would use a string difference checker.

We form a subset of all the strings from this step. The rest are stored in later statements, but before compilation, the compiler builds a derivation tree from the substrings it has captured from the input. The stems are either non-string values cast into strings or strings we have already defined.

Each branch represents a string in the output text generated from the above operations. If the stem does not contain a sequence in the output text, it is stored in the global object as an encoded string. The compiler does a depth-first travel of this tree and generates statements that map to the stored values. A copy of this tree exists during encoding.

Assignment _expressions_ are also allowed and return their result, so a statement like `x={x:x.y=1}` assigns two properties `x.x` and `x.y`, which both equal `1`; `x.y` is assigned before `x.x`, because expressions are evaluated from the inside. However, assignment is a _destructive_ operation, so the compiler does not override any keys to avoid breaking the output.

#### Symbols

We can represent sequences of symbols literally as strings without us performing any encoding or storing in the global object to be decoded or derived. However, we can also perform one additional optimization: minimizing backslashes.

If a substring contains many backslashes, then there are two options. The first is through a `RegExp` literal and calling `toString`, which yields a string of the pattern between two slashes without using the `source` property. Alternatively, we could also use the `String.raw` function.

However, if the compiler evaluates it, with native `eval` as a syntax error, despite using both the `replace` and `slice` methods, it falls back to using the normal substrings.

#### Integers

From here on out, many of the encodings, in principle, use bijective numeration, with `Number` or `BigInt` as an intermediate data type during conversion. All substrings are encoded in **bijective base 32** using a specific order of characters except otherwise specified. Again, any arbitrary sequence of digits from a given set has a single numeric representation.

Numeric-only substrings are decoded into `BigInt` and embedded. Zero padding is done as an additional step for substrings with leading zeroes.

#### Alphanumeric substrings

Alphanumeric substrings follow the same rules as numbers, except this time parsed as a number with a base higher than 10, depending on the position of the last letter in the substring.

By default, `toString()` yields lowercase. So if the original substring contains uppercase letters, then the positions of those characters in the substring will be converted into uppercase using compressed ranges. If the entire string is uppercase, then the string would be converted directly into uppercase.

#### Words with diacritics

For multilingual texts, use the Latin alphabet, along with a number of non-ASCII letters embedded inside. The initial substring is stripped of these non-ASCII characters and then encoded as a big integer (see above section). The non-ASCII characters are stored and encoded at the end of the string. Each insertion point consists of a substring and an insertion index. No Unicode normalization is performed, even if part of the encoding or decoding process.

#### Other writing systems and languages

For substrings of a different Unicode script other than Latin, they are generated from a pool of characters from the script. The result is encoded using the pool of characters into a big integer, or in the case of extremely long words or CJK text, _arrays_ of big integers.

For bicameral scripts like Cyrillic, Greek, Armenian and Georgian, the substring is encoded initially as lowercase and then a separate procedure converts selected characters into uppercase based on the input string.

#### Arbitrary Unicode sequences

For other Unicode characters, including CJK, special, private-use, non-printable and spacing characters, and even astral code points, they are converted from their hexadecimal values and grouped into smaller subsequences based on their leading digits. All leading zeroes are stripped when encoded and added back when decoded.

This yields pairs of symbol sequences: the "keys" being the leading and the "values" being the trailing digits that are not encoded. Both key and value pairs are converted from bijective base 16 into bijective base 30. `,` and `:` are reserved for separating keys and values.

## Customization

Here's a list of customization options available:

- `globalVar` - the global variable defined to store the encoded values. Must be a valid, undefined JavaScript identifier. Default is `$`.
- `resultVar` - the global variables to store the output string. Must be a valid, undefined JavaScript identifier. Default is `_`.
- `strictMode` - Includes a `var` or `let` declaration, setting it at the beginning of the program. Default is `null`, which does not include a declaration.
- `export` - Which key to export the string, if `moduleExports` is set to true. Default is `result`.
- `defaultQuote` - Quoting style to fall back to, if smart quoting is enabled. One of `single`, `double` or `backtick`. Default is `double`.
- `objectQuote` - Whether to quote keys inside objects, and which quotes to use. `none` skips quoting identifier keys, so sequences of `_` and `$` will not be quoted. If `calc` is selected then all the keys would be quoted inside square brackets. Default is `none`. One of `none`, `single`, `double` or `calc`.
- `smartQuote` - Whether or not to enable smart quoting; choosing quotes which have the least number of escapes. If disabled, all strings inside the output including object keys will be quoted to `defaultQuote` and `objectQuote`. Default is `true`.
- `intThreshold` - Maximum length of decoded `BigInt's. If the length of the decoded `BigInt`is greater than this value, arrays of bijective base-**31**-encoded`BigInt's are used. Default is `200`.
- `delimiter` - Which character to use to delimit `BigInt`-encoded substrings or character sets. Default is `,`.
- `rangeDelimiter` - Which character to use to delimit ranges that `BigInt`-encoded character sets. Default is `-`.
- `wrapInIIFE` - Whether to wrap the obfuscated code in an anonymous function call. Uses the `Function` constructor. Default is `false`.
- `logResult` - Whether or not to log the obfuscated code in the console. Default is `false`.

## FAQs

#### Why would I want to obfuscate my text?

There are many reasons why it's a good idea to protect your work, so to prevent anyone from copying or pasting your work. This is especially important on private work, such as manuscripts for novels, personal or sensitive information, or even code like s client-side games or command-line interfaces,

#### Is this obfuscator absolutely foolproof?

The generated code "decrypts" or de-obfuscates itself only when run in a Node.JS environment, so it can only be considered a step in the process if you really want maximum privacy.

This generated code, including the header, is programmatically generated from its parameters. And because there is a one to one correspondence between the sequence of substrings in the input and output, the source can obviously be recovered and reverse engineered, so it may not be obvious.

#### Why is my obfuscated code larger than my original source?

Because there are only 32 different kinds of characters in the output, the ratio of input to output depends heavily on which characters are in the string and how often they occur together.

Sequences of any of these 32 symbol or punctuation characters are encoded literally in the string, they get a 1-to-1 encoding except for escape sequences `\`, `\", `\"` and "\` ". Spaces have a 1-to-1 correspondence because they are encoded as commas.

Words, numbers and other alphanumerics are encoded once and stored in the global object, so any repeat sequence of any of these would only be encoded as a property of the global object.

You don't have to worry too much about code size because there is a lot of repetition and only 32 characters.

#### Can I run a minifier or prettifier such as Prettier or UglifyJS on the obfuscated output?

Yes. For most cases, like small inputs of probably a few thousand words. Since there are many characters in the output, and perhaps many tokens in the output, it would probably break your formatter or minifier or whatever is used to display your result, if your text is more than a million characters long.

Source: During development, this was tested on minified source code with Prettier with its plugins.
