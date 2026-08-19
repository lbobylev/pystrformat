# pystrformat

`pystrformat` is a flexible and extensible string-formatting library for Haskell.

It provides formatting inspired by Python's `str.format()` syntax while keeping
the formatting engine open to custom syntaxes, variable containers, and
user-defined formatting rules.

## Features

`pystrformat` supports:

- Automatically numbered placeholders: `{}`;
- Positional placeholders: `{0}`, `{1}`;
- Named placeholders: `{name}`;
- Placeholders in arbitrary order;
- Reusing the same variable multiple times;
- Ignoring variables that are not referenced by the format string;
- Per-value format specifications, such as `{0:+8.4}`.

The library favors **functionality and extensibility** over having the smallest
possible implementation or maximizing performance in every use case.

## Format strings

Formatting specifications are represented by the `Format` type.

A `Format` value can be parsed from lazy `Text`. Since `Format` implements
`IsString`, format strings can also be written directly as string literals.

For example:

    format "Hello, {name}!" variables

## Formatting syntaxes

The package provides two built-in formatting syntaxes.

### Python-like syntax

This is the default syntax and the one used by the `IsString Format` instance.

Anything enclosed in braces is interpreted as a variable substitution:

    {}
    {0}
    {name}
    {0:+8.4}

The syntax is intentionally similar to Python's `str.format()`.

### Shell-like syntax

An alternative shell-inspired syntax is also available, where variable
substitutions are introduced with a dollar sign.

## Custom syntaxes

The formatting syntax is not hard-coded into the rest of the library.

Custom syntaxes can be implemented by parsing input into values of the `Format`
type. This allows applications to provide their own notation while continuing to
use the same formatting engine.

## Variable containers

The `format` function takes a `Format` specification and a container of
variables.

Variable containers are generalized by the `VarContainer` type class.

Built-in containers include:

- `Single` for formatting a single value;
- tuples;
- lists;
- `[(Text, a)]`;
- `Map Text a`.

Tuples and lists provide positional variables:

    {0}
    {1}
    {2}

Association lists and maps provide named variables:

    {name}
    {count}
    {value}

Applications can define their own `VarContainer` instances as well. For example,
a record type can expose its fields directly as formatting variables.

## Formattable values

Values that can participate in substitutions are generalized through the
`Formatable` type class.

A `Formatable` instance defines:

- how a value is formatted by default;
- which format specifications the value understands.

The built-in instances cover common Haskell types, including:

- integers:
  - `Int`
  - `Integer`
  - `Int8` through `Int64`
  - `Word8` through `Word64`
- floating-point values:
  - `Float`
  - `Double`
- strings and text:
  - `String`
  - strict and lazy `Text`
  - strict and lazy `ByteString`
- `Bool`
- date and time values from `Data.Time`

Additional numeric types can be supported by providing suitable instances.

Any value with a `Show` instance can also be formatted by wrapping it in the
`Shown` constructor.

Custom application types can implement `Formatable` directly to provide their
own formatting behavior and format-specification syntax.

## Extensibility

The library is designed so that the main pieces of the formatting system can be
replaced or extended independently.

You can define:

- custom format-string syntaxes;
- custom variable containers through `VarContainer`;
- custom value formatting through `Formatable`.

This makes `pystrformat` suitable not only as a Python-style formatting library,
but also as a foundation for application-specific formatting and templating
conventions.

## Examples

See the `examples/` directory and Haddock documentation for usage examples.

## License

BSD-3-Clause.

See the `LICENSE` file for the complete license terms and copyright notices.
