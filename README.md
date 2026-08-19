# pystrformat

`pystrformat` is a continuation of [`text-format-heavy`](https://github.com/portnov/text-format-heavy),
originally created by Ilya Portnov, and provides Haskell string formatting inspired by Python's `str.format()` syntax.

It supports positional and named placeholders, per-value format specs, custom
variable containers, and custom `Formatable` instances.

## Quick Start

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Data.Text.Format.Heavy
import qualified Data.Text.Lazy as TL

hello :: TL.Text
hello = format "Hello, {name}!" [("name", "world" :: TL.Text)]
```

## Placeholder Syntax

```haskell
format "{}" (Single ("hello" :: TL.Text))
-- "hello"

format "{0} {1}" ("hello" :: TL.Text, "world" :: TL.Text)
-- "hello world"

format "{name}" [("name", "world" :: TL.Text)]
-- "world"

format "{{name}}" ()
-- "{name}"
```

The default syntax uses braces. Literal braces can be escaped as `{{` and `}}`.

## Passing Variables

Use `Single` for one value:

```haskell
format "value: {}" (Single (42 :: Int))
-- "value: 42"
```

Use tuples or lists for positional placeholders:

```haskell
format "{0}, {1}" ("hello" :: TL.Text, "world" :: TL.Text)
-- "hello, world"

format "{0}, {2}" (Several ["zero", "one", "two" :: TL.Text])
-- "zero, two"
```

Use association lists or maps for named placeholders:

```haskell
format "Hello, {name}!" [("name", "Alice" :: TL.Text)]
-- "Hello, Alice!"
```

For heterogeneous named values, wrap each value in `Variable`:

```haskell
let vars =
      [ ("name", Variable ("Alice" :: TL.Text))
      , ("count", Variable (3 :: Int))
      ]

format "{name} has {count} messages" vars
-- "Alice has 3 messages"
```

## Format Specs

Format specs are written after `:` and interpreted by the value being formatted:

```haskell
format "hex: {:#x}" (Single (427 :: Int))
-- "hex: 0x1ab"

format "float: {:+6.4}" (Single (2.718281828 :: Double))
-- "float: +2.7183"

format "center: <{:^10}>" (Single ("hello" :: String))
-- "center: <   hello  >"

format "upper: {:~u}" (Single ("hello" :: TL.Text))
-- "upper: HELLO"

format "bool: {:yes:no}" (Single False)
-- "bool: no"
```

`Maybe` values can specify a fallback after `|`:

```haskell
format "value: {:.3|<missing>}" (Single (Nothing :: Maybe Float))
-- "value: <missing>"
```

Any value with a `Show` instance can be formatted through `Shown`:

```haskell
format "debug: {}" (Single (Shown (Just True)))
-- "debug: Just True"
```

## Error Handling

`format` is convenient and throws an error if formatting fails.

Use `formatEither` when errors should be handled explicitly:

```haskell
import Data.Text.Format.Heavy.Build (formatEither)

formatEither "missing: {1}" (Single ("value" :: TL.Text))
-- Left "Parameter not found: 1"
```

## Parsing And Introspection

Use `Data.Text.Format.Heavy.Parse.parse` to inspect a format string without
formatting it:

```haskell
import qualified Data.Text.Format.Heavy.Parse as Format

Format.parse "{name}"
-- Right [FormatReplacementField "name" Nothing]

Format.parse "{name:.2}"
-- Right [FormatReplacementField "name" (Just ".2")]

Format.parse "{{name}} {name}"
-- Right [FormatString "{name} ", FormatReplacementField "name" Nothing]

Format.parse "{value:{width}}"
-- Right [FormatReplacementField "value" (Just "{width}")]
```

This API exposes the field name and the raw format spec parsed from the default
braces syntax. It does not format values.

## Shell-Like Syntax

The default syntax uses braces. A shell-like syntax is also available:

```haskell
import Data.Text.Format.Heavy
import Data.Text.Format.Heavy.Parse.Shell
import qualified Data.Text.Lazy as TL

format (parseShellFormat' "Hello, $name!") [("name", "world" :: TL.Text)]
-- "Hello, world!"
```

Braced shell-style placeholders can also carry format specs:

```haskell
format (parseShellFormat' "hex: ${:#x}") (Single (427 :: Int))
-- "hex: 0x1ab"
```

## Extending

Applications can extend the library by defining:

- `Formatable` instances for custom value types.
- `VarContainer` instances for custom variable sources.
- Custom parsers that produce `Format`.

## License

BSD-3-Clause. See `LICENSE`.
