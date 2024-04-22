# Mojo standard library development

This document covers the essentials of developing for the standard library.

## Prerequisites

If this is your first time contributing, first read everything in
[CONTRIBUTING.md](../../CONTRIBUTING.md#fork-and-clone-the-repo). Logistically,
you need to do the following:

1. [Fork and clone the repo](../../CONTRIBUTING.md#fork-and-clone-the-repo)
2. [Branch off nightly](../../CONTRIBUTING.md#branching-off-nightly)
3. [Install the nightly Mojo compiler](../../CONTRIBUTING.md#getting-the-nightly-mojo-compiler)

And if you're using VS Code:

- [Install the nightly VS Code
  extension](https://marketplace.visualstudio.com/items?itemName=modular-mojotools.vscode-mojo-nightly)

## Building the standard library

To build the standard library, you can run the
[`build-stdlib.sh`](../scripts/build-stdlib.sh) script from the
`mojo/stdlib/scripts/` directory. This will create a build artifacts directory,
`build`, in the top-level of the repo and produce the `stdlib.mojopkg` inside.

```bash
./stdlib/scripts/build-stdlib.sh
```

## Testing the standard library

### Installing unit test dependencies

To run the unit tests, you first need to install `lit`:

```bash
python3 -m pip install lit
```

And make sure that `FileCheck` from LLVM is on path. If your are on macOS, you
can `brew install llvm` and add it to your path in `~/.zshrc` or `~/.bashrc`:

```bash
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```

If you are on Ubuntu you can:

```bash
sudo apt update

sudo apt install llvm
```

And it will be available in `PATH`.

In the near future, we will be moving away from `FileCheck` in favor of writing
the unit tests using our own `testing` module and remove this dependency
requirement for contributors. We are happy to welcome contributions in this
area!

### Running the standard library tests

We provide a simple Bash script to build the standard library package and
`test_utils` package that is used by the test suite.

Just run `./stdlib/scripts/run-tests.sh` which will produce the necessary
`mojopkg` files inside your `build` directory, and then run `lit -sv
stdlib/test`.

```bash
./stdlib/scripts/run-tests.sh

lit -sv stdlib/test
```

All the tests should pass on the `nightly` branch with the nightly Mojo
compiler. If you've pulled the latest changes and they're still failing please
[open a GitHub
issue](https://github.com/modularml/mojo/issues/new?assignees=&labels=bug%2Cmojo&projects=&template=mojo_bug_report.yaml&title=%5BBUG%5D).

### Running a subset of the Standard Library Unit Tests

If you’d like to run just a subset of the tests, feel free to use all of the
normal options that the `lit` tool provides.  For example, to run just the
`builtin` and `collections` tests:

```bash
lit -sv stdlib/test/builtin stdlib/test/collections
```

This can quickly speed up your iteration when doing development to avoid running
the entire test suite if you know your changes are only affecting a particular
area. We recommend running the entire test suite before submitting a Pull
Request.

Reminder that if you’re choosing to invoke `lit` directly and not use the
`run-tests.sh`, you need to ensure your `stdlib.mojopkg` and
`test_utils.mojopkg` are up-to-date. We’re not currently imposing any build
system right now to ensure these dependencies are up-to-date before running the
tests.

If you run into any issues when running the tests,
[please file an issue](https://github.com/modularml/mojo/issues) and we’ll take
a look.

## Formatting changes

Please make sure your changes are formatted before submitting a pull request.
Otherwise, CI will fail in its lint and formatting checks.  The `mojo` compiler
provides a `format` command.  So, you can format your changes like so:

```bash
mojo format <file1> <file2> ...
```

You can also do this before submitting a pull request by running it on the
relevant files changed compared to the remote:

```bash
git diff origin/main --name-only -- '*.mojo' | xargs mojo format
```

You can also consider setting up your editor to automatically format
Mojo files upon saving.

## Tutorial

Here is a complete walkthrough, showing how to make a change to the Mojo
standard library, test it, and raise a PR.

First, follow everything in the [prerequisites](#prerequisites).

__IMPORTANT__ We'll be in the `mojo/stdlib` folder for this tutorial, check and
make sure you're in that location if anything goes wrong:
`cd [path-to-repo]/stdlib`

### A simple change

Let's try adding a small piece of functionality to `path.mojo`:

```mojo
# ./stdlib/src/pathlib/path.mojo

fn print_cwd() raises:
    print("cwd:", cwd())
```

And make sure you're importing it from the module root:

```mojo
# ./stdblib/src/pathlib/__init__.mojo

from .path import (
    DIR_SEPARATOR,
    cwd,
    print_cwd,
    Path,
)
```

Now you can create a temporary file named `main.mojo` for trying out the new
behavior. You wouldn't commit this file, it's just to experiment with the
functionality before you write tests:

```mojo
# ./stdlib/main.mojo

from src import pathlib

def main():
    pathlib.print_cwd()
```

Now when you run `mojo main.mojo` it'll reflect the changes:

```plaintext
cwd: /Users/jack/src/mojo/stdlib
```

### A change with dependencies

Here's a more tricky example that modifies multiple standard library files that
depend on each other.

Try adding this to `os.mojo`, which depends on what we just added to
`pathlib.mojo`:

```mojo
# ./stdlib/src/os/os.mojo

import pathlib

fn print_paths() raises:
    pathlib.print_cwd()
    for path in cwd().listdir():
        print(path[])
```

This won't work because it's importing `pathlib` from the `stdlib.mojopkg` that
comes with Mojo. We'll need to build our just-modified standard library running
the command:

```bash
./scripts/build-stdlib.sh
```

This builds the standard library and places it at the root of the repo in
`../build/stdlib.mojopkg`. Now we can edit `main.mojo` to use the normal import
syntax:

```mojo
import os

def main():
    os.print_paths()
```

We also need to set the following environment variable that tells Mojo to
prioritize imports from the standard library we just built, over the one that
ships with Mojo:

```bash
MODULAR_MOJO_NIGHTLY_IMPORT_PATH=../build mojo main.mojo
```

Which now outputs:

```plaintext
cwd: /Users/jack/src/mojo/stdlib
main.mojo
test
docs
scripts
src
```

### Bonus tip: fast feedback loop

If you like a fast feedback loop, try installing the file watcher `entr`, which
you can get with `sudo apt install entr` on Linux, or `brew install entr` on
macOS. Now run:

```bash
export MODULAR_MOJO_NIGHTLY_IMPORT_PATH=../build

ls **/*.mojo | entr sh -c "./scripts/build-stdlib.sh && mojo main.mojo"
```

Now, every time you save a Mojo file, it packages the standard library and
runs `main.mojo`.

### Running tests

If you haven't already, follow the steps at:
[Installing unit test dependencies](#installing-unit-test-dependencies)

### Adding a test

This will show you how the `FileCheck` utility works. First, turn it on by
adding it to the end of line 7 in `./stdlib/test/pathlib/test_pathlib.mojo`:

```mojo
# RUN: %mojo -debug-level full -D TEMP_FILE=%t %s | FileCheck %s
```

Now we can add the test and force it to fail:

```mojo
# CHECK-LABEL: test_print_cwd

def test_print_cwd():
    print("== test_print_cwd")

    # CHECK: This will fail
    print_cwd()
```

Don't forget to call it from `main()`:

```bash
def main():
    test_print_cwd()
```

Now, instead of testing all the modules, we can just test `pathlib`:

```bash
lit -sv test/pathlib
```

This will give you an error showing exactly what went wrong:

```plaintext
/Users/jack/src/mojo-toy-2/stdlib/test/pathlib/test_pathlib.mojo:27:11:
error: CHECK: expected string not found in input

# CHECK: This will fail
          ^
<stdin>:1:18: note: scanning from here
== test_print_cwd
                 ^
```

Lets fix the test that we just added:

```plaintext
# CHECK-LABEL: test_print_cwd

def test_print_cwd():
    print("== test_print_cwd")

    # CHECK: cwd:
    print_cwd()
```

We're now checking that `print_cwd` is prefixed with `cwd:` just like the
function we added. Run the test again:

```plaintext
Testing Time: 0.65s

Total Discovered Tests: 1
  Passed: 1 (100.00%)
```

Success! Now we have a test for our new function.

The last step is to [run mojo format](#formatting-changes) on all the files.

### Raising a PR

Make sure that you've had a look at all the materials from the standard library
[README.md](../README.md). This change wouldn't be accepted because it's missing
tests, and doesn't add useful functionality that warrants new functions. If you
did have a worthwhile change you wanted to raise, follow the steps to
[create a pull request](../../CONTRIBUTING.md#create-a-pull-request).

Congratulations! You've now got an idea on how to contribute to the standard
library, test your changes, and raise a PR.

If you're still having troubles make sure to reach out on
[GitHub](https://github.com/modularml/mojo/discussions/new?category=general) or
[Discord](modul.ar/discord)!
