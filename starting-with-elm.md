# Starting with Elm

Elm is a functional programming language for declaratively creating web
browser-based graphical user interfaces. Elm uses the functional reactive
programming style and purely functional graphical layout to build user
interface without any destructive updates.

## The elm tool-chain

Elm comes with the `elm` command line tool that is installable via `brew` in
MacOS:

```bash
$> brew install elm
```

`elm` provides the following subcommands:

### elm-repl

REPL stands for Read-eval-print-loop and it is an interactive shell for the
Elm programming language - much like `irb` and `IPyhon`.

### elm-reactor

The Elm reactor is a development server the compiles Elm applications into
HTML web pages on the fly. You can set the address and port it will listen to
and then use a browser the try out an Elm application. The reactor will re-
compile the application when the page is reloaded.

### elm-package

The `elm package` command is Elm's package manager. Each Elm package has the
following structure:

```
my-awesome-elm-pkg/
	elm-stuff/            Directory that contains the Elm depedencies
	elm-package.json      Meta-file that defines the package and its dependencies
	*.elm                 Elm source code files
```

* To bootstrap an Elm project run `elm package install`. This will generate
and `elm-package.json` and it will instal Elm's core package.
* To re-install the dependencies of an existing package run `elm package
install` as well. It will read the `elm-package.json` and install the required
packages.
* To add a dependency in a project run `elm packgae install <Package name>`.
This will install the requested package and properly modify `elm-package.json`.

### elm-make

The `elm make` command is the compiler of Elm. It supports two ways of
compiling an application.

For a full screen application when Elm is taking care of all the HTML and CSS
management, run `elm make Source.elm --output index.html`. This will compile
`Source.elm` file into HTML, CSS and JavaScript.

For a widget or a JavaScript library, run `elm make Source.elm --output
bundle.js`. This will compile Elm in JavaScript and you will be responsible for
writing the HTML and CSS to make up the application.

## The Elm programming language

Elm is a strongly-typed functional programming language. Hence, Elm functions
are pure functions. This entails the following:

* The function always evaluates the same result value given the same argument
value(s). The function result value cannot depend on any hidden information or
state that may change while program execution proceeds or between different
executions of the program, nor can it depend on any external input from I/O
devices.
* Evaluation of the result does not cause any semantically observable side
effect or output, such as mutation of mutable objects or output to I/O devices.

The function definition looks like the following:

```elm
isNegative n =
	if n < 0 then
		True
	else
		False
```

### Data structures

Elm offers lists, tuples and records as the primitive data structures.

#### List

Lists hold multiple instance of the same type. Lists in Elm support the
following operations:

* Concatenation: `[1,2,3] ++ [3,4,5] == [1. 2. 3. 3. 4. 5]`
* Map: `List.map isNegative [-1, 1, 3, -2] == [True, False, False, True]`
* Reduce: `add x y = x + y; List.foldr add 0 [1, 2, 3] == 6`
* Sorting: `List.sort [3, 1, 2] == [1, 2, 3]`
* ...

#### Tuple

Tuples hold a fixed number of values of different types.

```elm
me =
	("George L.", 25)
```

#### Record

A record is a set of key-value pairs, similar to objects in JavaScript and
dictionaries in Python.

```elm
me =
	{name = "George L.", age = 25}
```

To access the property you can use `.<Property>` function as follows:

```elm
george = me
george.name == "George L."
.age george == 25
```

And since the property accessor is a function, you can treat it like a function:

```elm
List.map .name [{name = "George L.", age = 25}, {name = "Frank U.", age = 26}] == ["George L.", "Frank U."]
```

You can use the properties in functions:

```elm
isGeorge {name} =
	if name == "George  L." then
		True
	else
		False
isGeorge {name = "George L.", age = 25} == True
isGeorge {name = "Frank U.", age = 26} == False
```

Finally, updating a property will create a new record as updates are not
destructive:

```elm
george = me
{george | age = 26} == {name = "George L.", age = 26}
```

Records are similar to JavaScript objects but there are fundamental differences. Namely:

* You cannot access fields that do no exist
* No field is ever null or undefined
* Records are separated from functions

Elm records support structural typing which means that records do not make up
static types. This allows for records to be used in any appropriate situation
as long as the necessary fields exist:

```elm
isItalian {origin} =
	if origin == "Italy" then
		True
	else
		False

isMidfielder {position} =
	if position == "midfielder" then
		True
	else
		False

maldini = {name = "Maldini", origin = "Italy", position = "defender"}
isItalian maldini == True
isMidfielder maldini == False
```

## References

1. https://en.wikipedia.org/wiki/Elm_(programming_language)
2. http://elm-lang.org/get-started
3. https://en.wikipedia.org/wiki/Pure_function
4. http://elm-lang.org/guide/core-language
