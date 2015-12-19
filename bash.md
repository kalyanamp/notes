# Notes on BASH

BASH is a very useful language for anyone who has to deal with operations and
application/system/cluster deployment. However unless you spend each day of
your day job writting BASH, it is easy to forget some of the unusual
differences of BASH with other languages. This list of gotchas is meant to be
a reminder of constructs I tend to forget all the time when I have to revisit
BASH after a while.

## Functions

There are three ways of defining functions in BASH. The first two are using
the `function` keyword:

```bash
function aFunc() {
  # Function body
}

function aFunc {
  # Function body
}
```

The third one is not using the `function` keyword, but makes the empty
brackets mandatory. The advantage of this style is that is portable since it
works with other shells (e.g.: `sh`). The reason being that this is part of
the [POSIX standard][PosixShell].

```bash
aFunc() {
  # Function body
}
```

### Function parameters

Function parameters work similarly with command line arguments. Bellow is a
function that expects three arguments. The last one is optional and it 
has a default value:

```bash
registerContact() {
  [ $# -lt 2 ] && echo "Please specify name and phone!" && return
  name=$1
  phone=$2
  phoneBook=${3:-"default"}

  echo "Adding '$name' (tel.: $phone) in phone book $phoneBook"
}

# Call examples
registerContact
#   "Please specify name and phone!"
registerContact "George" "123"
#   "Adding 'George' (tel.: 123) in phone book default"
registerContact "George" "123" "friends"
#   "Adding 'George' (tel.: 123) in phone book friends"
```

In the previous example it is demonstrated how to exit from a function using
the `return` keyword. Also I showed how to call a function in BASH, and at
this point the resemblence with calling actual executables is very high.

### Returning/collecting output from a function

Functions in BASH can return by `echo`ing strings. This strings will be
forwarded to the stdout stream unless the function is called inside a subshell.
Additionally the `return` keyword can be used with a numeric exit code. This
call will not terminate the programm (like `exit`) but it will set the `$?`
variable the the specified number:

```bash
terminateProcess() {
  [ $# -ne 1 ] && echo "pid not specified" && return 1
  pid=$1
  kill -s term $pid
  [ $? -ne 0 ] && echo "failed to temrinate process $pid" && return 1
}

# Successful call
terminateProcess 123
#   $? is 0

# Failling call
err=$(terminateProcess)
if [ $? -ne 0 ]; then
  echo "Terminating server: $err"
fi
#   "Terminating process: pid not specified"
```

## References

[PosixShell]: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html "POSIX standard for Shell"

1. [Posix Shell standard][PosixShell]
