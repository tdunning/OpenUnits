# OpenUnits

The goal of the OpenUnits project is to provide an easy way to communicate the units associated with a quantity in an unambiguous, easily extensible, and machine-readable way.

Units in the sense here include the normal SI base and extended units as well as a small set of US and UK traditional measures, but also has extension points for chemical substances, currency and user defined types. These extensions are clearly not "units" in the sense of metrology and including them will, most likely, irk purists, but they are very useful in practice for, say, sustainability where `Mg {chem: CO2}` or `{currency: USD} / (kW h)` are useful.

OpenUnits only provides parsers and generators for a standard unit representation and is intended to work with a native units system in whatever computer language you are using. Dimensional analysis and unit conversion are out of scope. Thus, you might use [Unitful](https://painterqubits.github.io/Unitful.jl/stable/) in Julia to perform dimensional analysis or unit conversion but still interact with systems written in Python that use [pint](https://pint.readthedocs.io/en/stable/) or in Go using [go-units](https://github.com/bcicen/go-units).

# The syntax
At the lowest level, numbers, prefixed units with exponents are known as a `unit factor`. Unit factors are combined into unit expressions using multiplication (indicated by adjacency, possibly with intervening whitespace), division (indicated by a solidus) and grouping (indicated using parentheses `(` and `)`). Whitespace is generally not significant except to terminate factors (for instance `m m` is the same as `m^2` but `mm` is millimeters). To avoid ambiguity, multiplication and division are not allowed after a division. This requires that the unit of thermal conductivity be written using exponents `W m^-1 K^-1` or by using a parenthetical grouping as `W/(m K)`. Alternatives like `W/m/K`, `W/m K` or `W/m K^-1` are not allowed due to ambiguity (feedback welcome on this point).

Exponents are expressed as by using a caret `m s^-1`.

Formally, the BNF for this syntax is

```
unit-expression ::= <unit-factor>+ {"/" <denominator>}?
unit-factor ::= number | <prefixed-unit> | <other-mark>
number :== re/-?[0-9]+(\.[0-9]*)([eE]-?[0-9]+?/
prefixed-unit :== re/<prefix>*/ <unit>  <exponent>?
exponent :== "^" <number>
other-mark :== <chemical> | <currency> | <user-unit>
chemical :== re/{chem: [^}]+}/
currency :== "{currency: " <iso-currency-code> "}"
user-unit :== re/{[^}]*}/
denominator ::= <unit-factor> | "(" unit-expression ")"
```
White space is allowed between factors but is not required if the boundary between factors is unambiguous. Thus `3mm` is
equivalent to `3 mm`. Within a `<unit-factor>` ambiguity between a string of prefix characters and a defined unit is avoided by always requiring a unit. Thus `mm` is interpreted as `10^-3 meter` instead of `10^-6`. In the default definitions file, both grams and kilograms are units, but `kg` comes first so `Mkg` will be parsed as `10^6 kilogram` rather than `10^9 gram` and `kg` is interpreted as `1 kilogram` rather than `10^3 gram`. Similarly `m Wb` is meter Weber while `mWb` is milliWeber. Dimensionless quantities should be expressed as numbers. Thus, a frequency of 1 Hz would be written as `1/s`.

The ambiguity between chemicals, currencies and user-marks is always decided in favor of the first two. Possible values for `<prefix>`, `<unit>` and `<iso-current-code>` are taken from the definitions file.

Internally, implementations should use whatever character encoding is most idiomatic on any platform but ISO-Latin-1 is used to exchange unit expressions externally.

# How it works
All of the reference implementations provided here use a single machine readable file (called the definitions file) to specify all prefixes and units and their representations. Depending on the language, this file is translated into a form that can be incorporated into the code or is probed directly. The definitions file is provided in JSON for easy editing and easy parsing, but alternative forms could also provided for languages it that is more convenient. The JSON form is considered canonical all other formats should be created mechanically from it.

Parsing a unit expression will result in an abstract syntax tree which can be converted in an implementation specific fashion into an internal unit expression that is specific to the language being used. If the internal representation cannot represent the expression, an exception is raised. This can happen, for instance, if `5 {currency: USD}` is given but the internal representation cannot represent currency. 

The OpenUnits syntax is carefully designed to allow much of the work of parsing expressions to be done using regular expressions in order to simplify implementations.

# Common tests
There are a number of tests specified in `OpenUnits` to simplify building compatibility packages. The simplest tests consist of a string to be parsed, the resulting syntax tree expressed in Polish notation and the result of generating a string from that syntax tree. Either the input or output strings can be missing (but not both). Tests may also include a dimensional analysis of the expression which is a vector of seven exponents, one for each of the base SI units. Such dimensional expressions be in the form of a string with some of the dimensions time (T), length (L), mass (M), electric current (I), absolute temperature (Θ), amount of substance (N), luminous intensity (J), chemical compount (Ch) and currency (C) each concatenated with positive or negative exponents. Thus, `m/s^2` would have dimension `T-2L` and `{currency: USD}/(kg {chem: CO2})` (dollars per kilogram of carbon dioxide) would have dimension `CM-1Ch-1`. Dimension expressions can have the different dimensions in any order.

# Comparison to other systems
The [UCUM](https://github.com/ucum-org/ucum) system has similar goals as OpenUnits and has a much broader set of units. Unfortunately, UCUM is not open source since the license does not allow derivative works and has other field of use limitations. Limited ability to parse or produce strings in UCUM format may be added in the future, but it is explicitly not a goal to maintain or claim compatibility with UCUM. 

In addition, UCUM is only concerned with encoding standard metrological units. OpenUnits, on the other hand, allows for application-specific extensions as well as non-metrological units such as currencies.

Contributions to extend the units supported in OpenUnits are welcome but contributors should be sure that their contributions do not violate the restrictions on their sources. 

# Contributing
If you have special purpose units for your needs, you can always add a local definitions file and use that. If you think that your extensions would be broadly useful and the community agrees, then please do send a pull request with additional definitions.

If you want to support a new programming language, you can do that by pulling the definitions file from OpenUnits and writing your own parser and generator. It is good form to make sure that any implementation passes all of the common tests in this package and any limitations are documented. If you write a parser and generator, please send a pull request for the `README` file here so that we can link to your package.

You can also just send a pull request that integrates your parser implementation into OpenUnits. All new parser/generators should be well documented, pass the common tests and generally be acceptable to whatever package repository that is customary for the package in question.

Keep in mind that asking questions is a great way to contribute. There are no stupid questions.

In general, you should be sure that you have the right to make any contributions and be willing to have your contributions licensed under the Apache license. That license will allow others to change your implementation into whatever they want without necessarily giving their changes back or even making them available, so be clear that is OK with you.

# Community and governance
Currently, OpenUnits is governed using the [BDFL](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life) model with contributions very welcome. Anybody who makes more than a few quality contributions in the form of documentation, reviews, code or extensions will be granted committer rights. If a community of contributors forms, I will transition the project to a more community-driven governance model.

# Acceptable community behavior
Be kind. Be generous. 

For now, I will decide what that means and provide gentle nudges toward decorum. If repeated nudges fail, I am happy to exclude people, but would very much rather not do that. 

All contributors will be required to assign the copyright of their contribution to me or any successor organization that maintains this project. Because this project is released under a liberal open source license, you can always use any contributions freely.

# Help needed
We need help in the following areas
* additional unit definitions
* implementations in more languages, compatible with more unit packages
* Documentation
* More tests
* Better CI/CD frameworks and dashboards
* Questions and suggestions!
