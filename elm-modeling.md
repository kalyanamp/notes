# Modeling the problem in Elm

Elm provides several constructs to enable modeling of data with weird shapes.

## Contracts

Contracts are type annotations introduce before functions and variables:

```elm
isItalian : { record | origin : String} -> Bool
isItalian person =
	if (.origin person) == "Italy" then
		True
	else
		False
```

## Enumerations and state machines

An enumeration is a user type that has a predefined set of values it can take.
A state machine is an extension of the enumeration in the sense that is can also
hold values:

```elm
-- An enumeration
type BillType = New | Paid

-- A state machine
type User = Anonymous | LoggedIn String

username : User -> String
username user =
  case user of
    Anonymous ->
      "Anonymous"
    LoggedIn name ->
      name
```

Calling `username/1`:

* With anonymous: `username Anonymous == "Anonymous"`
* With a user: `u = LoggedIn "George"; username u == "George"`

## Tagged unions

Unions in C are data structures that can hold one piece of information of a
given type from a small set of types:

```elm
type Age = AgeStr (String) | AgeInt (Int)

age_str : Age -> String
age_str age =
  case age of
    AgeStr a ->
      a
    AgeInt a ->
      toString a
```

To run `age_str/2`:

* With a string age: `age = AgeStr "25"; age_str age == "25"`
* With a numeric age: `age = AgeInt 25; age_str age == "25"`

## Dealing with Null

Elm does not support a `null` value. However, it provides `Maybe` construct
that can be used as a type wrapper to tell callers that a function may or may
not return the expected type:

```elm
young : { record | age : Int, name : String } -> Maybe String
young person =
  if .age person < 20 then
    Just (.name person)
  else
      Nothing
```

## References

1. http://elm-lang.org/guide/model-the-problem
