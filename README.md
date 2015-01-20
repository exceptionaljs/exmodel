QData
=====

  * [Official Repository: jshq/qdata](https://github.com/jshq/qdata)
  * [Unlicensed] (http://unlicense.org)

QData is a high performance data processing and validation library based on an extensible schema builder. It allows to build a schema that can be then used to process and validate any kind of JavaScript data (the root variable can be object, array, or any other primitive type). The library is designed for critical systems where the performance is important and even a minor overhead at validation side can cause service delays. QData moves the data validation and processing into extreme by using a JavaScript code-generator that generates the best possible validation and data processing functions for any user-defined schema. Most of JavaScript engines today have built-in JIT compiler so the code generated by QData is basically JIT compiled by your JS engine and thus ultra-fast, outperforming all other JavaScript validation solutions.

The performance is not the only aspect and feature offered by QData. The library has been designed in a way that it should be very straightforward to define schema, to reuse or inherit the existing one, and to create your own types that will extend the built-in functionality as there is no library that could satisfy all possible use-cases, although QData tries hard to include the most important types as full-featured built-ins.

The schema structure is always declarative and most of the schemas can be serialized back to JSON (QData calls it a normalized JSON). The library also allows to associate a custom information called `metadata` with any field. Metadata is completely ignored by QData library, but other tools can take advantage of it (for example you can associate a SQL table names with your entities and use them in your DB layer).

Additionally, QData has several data processing options that help to deal with common problems like implementing data insertion, updating, deletion, and querying. Processing options can also be used to dismiss or filter out objects' properties that are not defined (useful when extracting information from request's body) and error accumulation (validation doesn't stop on the first error).

Disclaimer
----------

QData library has been designed to solve common, but also very specific problems. It's very fast and the support for metadata allows to simply extend it by new features. All built-in features are used in production and you will find many of them handy when implementing web services that do CRUD operations, because a single schema can be used to validate data that is inserted, updated, queried, or deleted. The library has been designed to be very fast, but is also very complete and configurable.

Introduction
------------

QData uses a declarative approach to build schemas, but it comes with its own syntax instead of relying on existing solutions like JSONSchema. The main reason for such move was to simplify the way schemas are defined by introducing shortcuts and directives that start with `$` character. Shortcuts are used to simplify declaration of the most common concepts (for example an array of integers can be written as `$type: int[]`) and directives are used to configure the type itself. Object's members are always defined without a `$` prefix, but it's possible to define also members that start with `$` by using escaping `$` as `\\$` (escaping and schema normalization is explained later).

```JS
var PersonSchema = qdata.schema({
  firstName  : { $type: "text", $maxLength: 64 },
  lastName   : { $type: "text", $maxLength: 64 },
  dateOfBirth: { $type: "date", $leapYear: false },

  active     : { $type: "bool" },
  score      : { $type: "int", $min: 0 },
  keywords   : { $type: "text[]" },
  bashrc     : { $type: "string", $maxLength: 4096 },

  address: {
    line1    : { $type: "text" },
    line2    : { $type: "text" },
    city     : { $type: "text" },
    zip      : { $type: "text" },
    country  : { $type: "text" }
  }
});
```

The example above defines a schema called `PersonSchema`, which is an `object` holding 8 properties of various types specified by `$type` directive. Careful readers have noticed that the root object and nested `address` object have omitted the `$type` directive. QData automatically uses `object` if no `$type` is provided, which allows to remove some verbosity in the schema declaration. Other directives like `$min`, `$max`, `$leapYear`, ..., are used to configure the type itself.

Confused by `string` vs `text` type? Well, `string` is _any_ string in JavaScript in contrast to `text`, which is a string that doesn't contain `\u0000-\u0008`, `\u000B-\u000C`, and `\u000E-\u001F` characters. These characters have special meaning and in many cases their presence in user data is unwanted and may be dangerous.

Confused by `[]` suffix in `keywords` member? That is one of QData shortcuts to define an array, which can also be defined by using `array` type like this:

```JS
var KeywordsSchemaLong = {
  $type: "array",
  $data: {
    $type: "text"
  }
};

var KeywordsSchemaShort = {
  $type: "text[]"
};
```

Both schemas defined above are equivalent and internally normalized into the same structure.

Data Processing Concepts
------------------------

QData comes with two base concepts that are used to work with data.

  - **`qdata.process(...)`** is a concept used to create a new data based on existing data. It's very useful in cases that more entities are mixed together in a single object and you need to separate/extract their content into independent objects. This happens for example in a request-body object. Data processing does not just validate the input data, but it can also sanity it before creating the output. If configured, you can trim/simplify input text, remove unknown properties, or insert fields having default values if they are not present.

  - **`qdata.test(...)`** is a concept used to test whether the given data conforms to the schema, but without using a sanitizer.

Built-In Data Types
-------------------

QData has a built-in support for the following data types:

Type and Aliases         | JS Type    | Description
:----------------------- | :--------- | :---------------------------------------
`bool`, `boolean`        | `boolean`  | Boolean
`double`, `number`       | `number`   | Double precision floating point number
`int8`                   | `number`   | 8-bit signed integer
`uint8`                  | `number`   | 8-bit unsigned integer
`int16`, `short`         | `number`   | 16-bit signed integer
`uint16`, `ushort`       | `number`   | 16-bit unsigned integer
`int32`                  | `number`   | 32-bit signed integer
`uint32`                 | `number`   | 32-bit unsigned integer
`int`, `integer`         | `number`   | 53-bit signed integer (safe integer)
`uint`                   | `number`   | 53-bit unsigned integer (safe integer)
`lat`, `latitude`        | `number`   | Latitude value (-90...90)
`lon`, `longitude`       | `number`   | Longitude value (-180...180)
`char`                   | `string`   | String containing exactly 1 character
`string`                 | `string`   | Any string
`text`                   | `string`   | Restricted string
`bigint`                 | `string`   | String containing a 64-bit integer as text
`color`                  | `string`   | A color values specified by hash `"#RRGGBB"` or a CSS name
`mac`                    | `string`   | MAC address
`ip`                     | `string`   | IPV4 or IPV6 address
`ipv4`                   | `string`   | IPV4 address
`ipv6`                   | `string`   | IPV6 address
`date`                   | `string`   | Date
`datetime`               | `string`   | Date and time without milliseconds
`datetime-ms`            | `string`   | Date and time with milliseconds (ms)
`datetime-us`            | `string`   | Date and time with microseconds (μs)
`object`                 | `object`   | Object type, default if `$type` is not specified
`array`                  | `array`    | Array type

Boolean
-------

Boolean `$type` is specified as `bool` or `boolean`.

Boolean type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$allowed`               | `bool[]`   | `null`  | Array of boolean values that are allowed. This is useful to restrict the value to be always `true` or `false`, but it does nothing if the array is empty or both `true` and `false` values are specified.

Number and Integer
------------------

Number type `$type` is specified by the following type names and properties:

Type and Aliases         | Minimum Value     | Maximum Value    | Description
:----------------------- | :---------------- | :--------------- | :-------------
`double`, `number`       | None              | None             | Double precision floating point
`int8`                   | -128              | 127              | 8-bit signed integer
`uint8`                  | 0                 | 255              | 8-bit unsigned integer
`int16`, `short`         | -32768            | 32767            | 16-bit signed integer
`uint16`, `ushort`       | 0                 | 65535            | 16-bit unsigned integer
`int32`                  | -2147483648       | 2147483647       | 32-bit signed integer
`uint32`                 | 0                 | 4294967295       | 32-bit unsigned integer
`int`, `integer`         | -9007199254740991 | 9007199254740991 | 53-bit signed integer, matches `Number.isSafeInteger()` behavior
`uint`                   | 0                 | 9007199254740991 | 53-bit unsigned integer, matches `Number.isSafeInteger()` behavior
`lat`, `latitude`        | -90               | 90               | Latitude (double precision)
`lon`, `longitude`       | -180              | 180              | Longitude (double precision)

Number type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$allowed`               | `number[]` | `null`  | Array of numbers that are allowed. If this directive is used it cancels all directives that specify minimum, maximum, or any other number related constraints
`$min`                   | `number`   | `null`  | Minimum value (the number has to be greater or equal than `$min`)
`$max`                   | `number`   | `null`  | Maximum value (the number has to be lesser or equal than `$max`)
`$gt`                    | `number`   | `null`  | Greater than (the number has to be greater than `$gt`)
`$lt`                    | `number`   | `null`  | Lesser than (the number has to be lesser than `$lt`)
`$divisibleBy`           | `number`   | `null`  | The number has to be divisible by this value (without a remainder)

Character
---------

Character `$type` is specified as `char` and it's a string that has length equal to one (that is, one character long string).

Character type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `false` | Specifies if the char can be an empty string
`$allowed`               | `char[]`   | `null`  | Array of characters that are allowed

String and Text
---------------

String `$type` is specified as `string` or `text`. If type `string` is specified any JavaScript string passes, however, if `text` type is specified the validator only passes if the string doesn't contain `\u0000-\u0008`, `\u000B-\u000C`, and `\u000E-\u001F` characters. Use `text` to disallow these characters that have special meaning and are in most cases unwanted (especially the `\u0000` character).

String type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$allowed`               | `string[]` | `null`  | Array of strings that are allowed. If this directive is used it cancels all directives related to string length validation, except `$empty` directive, which always applies, regardless of other constraints
`$length`                | `number`   | `null`  | Exact string length
`$minLength`             | `number`   | `null`  | Minimum string length
`$maxLength`             | `number`   | `null`  | Maximum string length
`$re`                    | `RegExp`   | `null`  | Regular expression

BigInt
------

BigInt `$type` is specified as `bigint`. BigInt is a string that contains only ASCII digits (characters from `0` to `9`) and an optional minus sign at the beginning. QData allows to validate whether the number represented as a string doesn't overflow 64 bits and also allows to set a possible minimum and maximum value (also as string). BigInt can also be configured to allow more than 64-bits by using `$min` and `$max` directives, described below.

BigInt type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$allowed`               | `string[]` | `null`  | Array of strings that are allowed. If this directive is used it cancels `$min` and `$max` constraints
`$min`                   | `string`   | `null`  | Minimum value (as string)
`$max`                   | `string`   | `null`  | Maximum value (as string)

Optionally, you can use `qdata.util.isBigInt(s, min, max)` function to check whether a string value matches BigInt with optional `min` and `max` constraints. This function doesn't require a schema instance.

Color
-----

Color `$type` is specified as `color`. Color is a string that matches `#RGB`, `#RRGGBB` or `color-name` format. QData supports all color names that are defined by CSS specification and allows to include a dictionary having extra color names that you need to allow. Color names are case-insensitive by default.

Color type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$cssNames`              | `bool`     | `true`  | Specifies if CSS color names are allowed
`$extraNames`            | `set[]`    | `null`  | A set (dictionary having `key: true`) that contains extra color names that are allowed

MAC Address
-----------

MAC address `$type` is specified as `mac`. MAC address is a string in form `XX:XX:XX:XX:XX:XX` that specifies a network MAC address.

MAC address type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$separator`             | `char`     | `:`     | Specifies separator used between MAC address components

IP Address
----------

IP address `$type` is specified as `ip`, `ipv4`, or `ipv6`. IP address is a string specifying a network IP address. The `ip` type matches both IPV4 and IPV6 addresses, in contrast to `ipv4` and `ipv6` types, that match IPV4 and IPV6 addresses, respectively.

IP address type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$allowPort`             | `bool`     | `false` | Specifies id port is also allowed

Date and DateTime
-----------------

Date+time `$type` is specified as `date`, `datetime`, `datetime-ms`, and `datetime-us`. It's a formatted string that contains date, time, or date+time components. QData validator extracts these components and validates whether they are correct. Leap years and leap seconds support is built-in and can be configured through directives.

Default date+time formats:

Date Type                | Format                       | Description
:----------------------- | :--------------------------- | :---------------------
`date`                   | `YYYY-MM-DD`                 | Date only
`datetime`               | `YYYY-MM-DD HH:mm:ss`        | Date+time
`datetime-ms`            | `YYYY-MM-DD HH:mm:ss.SSS`    | Date+time+ms
`datetime-us`            | `YYYY-MM-DD HH:mm:ss.SSSSSS` | Date+time+μs

Date+time format options:

Format Option            |Fixed Length| Range           | Description
:----------------------- | :----------| :-------------- | :---------------------
`Y`                      | `false`    | `1-9999`        | Year (1-4 digits)
`YY`                     | `true`     | `00-99`         | Year (2 digits)
`YYYY`                   | `true`     | `0001-9999`     | Year (4 digits)
`M`                      | `false`    | `1-12`          | Month (1-2 digits)
`MM`                     | `true`     | `01-12`         | Month (2 digits)
`D`                      | `false`    | `1-31`          | Day (1-2 digits)
`DD`                     | `true`     | `01-31`         | Day (2 digits)
`H`                      | `false`    | `0-23`          | Hour (1-2 digits)
`HH`                     | `true`     | `00-23`         | Hour (2 digits)
`m`                      | `false`    | `0-59`          | Minute (1-2 digits)
`mm`                     | `true`     | `00-59`         | Minute (2 digits)
`s`                      | `false`    | `0-60`          | Second (1-2 digits)
`ss`                     | `true`     | `00-60`         | Second (2 digits)
`SSS`                    | `true`     | `000-999`       | Millisecond (3 digits)
`SSSSSS`                 | `true`     | `000000-999999` | Microsecond (6 digits)
`?`                      | `true`     |                 | Any other character requires exact match of that character, for example `-`, `/`, `.`, `,`, etc...

Date+time type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$format`                | `string`   | `null`  | Specifies date+time format, see format options above
`$leapYear`              | `bool`     | `true`  | Specifies whether to allow leap year date
`$leapSecond`            | `bool`     | `false` | Specifies whether to allow leap second date+time

Object
------

Object `$type` is specified as `object` or can be omitted completely.

TODO

Object type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$undefined`             | `bool`     | `false` | Specifies if the value can be `undefined`

Array
-----

Array `$type` is specified as `array` or by `[]` suffix (shortcut).

TODO

Custom Types
------------

TODO

License
-------

QData follows [Unlicense](http://unlicense.org/).