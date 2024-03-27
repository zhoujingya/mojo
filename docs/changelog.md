# Mojo unreleased changelog

This is a list of UNRELEASED changes for the Mojo language and tools.

When we cut a release, these notes move to `changelog-released.md` and that's
what we publish.

[//]: # Here's the template to use when starting a new batch of notes:
[//]: ## UNRELEASED
[//]: ### ⭐️ New
[//]: ### 🦋 Changed
[//]: ### ❌ Removed
[//]: ### 🛠️ Fixed

## UNRELEASED

### 🔥 Legendary

- Tuple now works with memory-only element types like String.  Also, `Tuple.get`
  now supports a form that just takes an element index but does not
  require you to specify the result type.  Instead of `tup.get[1, Int]()` you
  can now just use `tup.get[1]()`.

### ⭐️ New

- Heterogenous variadic pack arguments now work reliably even with memory types,
  and have a more convenient API to use, as defined on the `VariadicPack` type.
  For example, a simplified version of `print` can be implemented as:

  ```mojo
  fn print_nicely(**kwargs: Int) raises:
    for key in kwargs.keys():
        print(key[], "=", kwargs[key[]])

   # prints:
   # `a = 7`
   # `y = 8`
  print_nicely(a=7, y=8)
  ```

  There are currently a few limitations:
  - The ownership semantics of variadic keyword arguments are always `owned`.
    This is applied implicitly, and cannot be declared otherwise:

    ```mojo
    # Not supported yet.
    fn borrowed_var_kwargs(borrowed **kwargs: Int): ...
    ```

  - Functions with variadic keyword arguments cannot have default values for
    keyword-only arguments, e.g.

    ```mojo
    # Not allowed yet, because `b` is keyword-only with a default.
    fn not_yet(*, b: Int = 9, **kwargs: Int): ...

    # Okay, because `c` is positional-or-keyword, so it can have a default.
    fn still_works(c: Int = 5, **kwargs: Int): ...
    ```

  - Dictionary unpacking is not supported yet:

    ```mojo
    fn takes_dict(d: Dict[String, Int]):
      print_nicely(**d)  # Not supported yet.
    ```

  - Variadic keyword parameters are not supported yet:

    ```mojo
    # Not supported yet.
    fn var_kwparams[**kwparams: Int](): ...
    ```

- Added new `collections.OptionalReg` type, a register-passable alternative
  to `Optional`.

  - Doc string code blocks can now `%#` to hide lines of code from documentation
    generation.

    For example:

    ```mojo
    var value = 5
    %# print(value)
    ```

    will generate documentation of the form:

    ```mojo
    var value = 5
    ```

    Hidden lines are processed as if they were normal code lines during test
    execution. This allows for writing additional code within a doc string
    example that is only used to ensure the example is runnable/testable.

  - Doc string code blocks can now `%#` to hide lines of code from documentation
    generation.

    For example:

    ```mojo
    var value = 5
    %# print(value)
    ```

    will generate documentation of the form:

    ```mojo
    var value = 5
    ```

    Hidden lines are processed as if they were normal code lines during test
    execution. This allows for writing additional code within a doc string
    example that is only used to ensure the example is runnable/testable.

### 🦋 Changed

- Mojo now warns about unused values in both `def` and `fn` declarations,
  instead of completely disabling the warning in `def`s.  It never warns about
  unused `object` or `PythonObject` values, tying the warning to these types
  instead of the kind of function they are unused in.  This will help catch API
  usage bugs in `def`s and make imported Python APIs more ergonomic in `fn`s.

- The [`DynamicVector`](/mojo/stdlib/collections/vector.html#dynamicvector) and
  [`InlinedFixedVector`](/mojo/stdlib/collections/vector.html#inlinedfixedvector)
  types now support negative indexing. This means that you can write `vec[-1]`
  which is equivalent to `vec[len(vec)-1]`.

- The [`isinf()`](/mojo/stdlib/math/math#isinf) and
  [`isfinite()`](/mojo/stdlib/math/math#isfinite) methods have been moved from
  `math.limits` to the `math` module.

- The `ulp` function has been added to the `math` module. This allows one to get
  the units of least precision (or units of last place) of a floating point
  value.

- `EqualityComparable` trait now requires `__ne__` function for conformance in addition
  to the previously existing `__eq__` function.

- Many types now declare conformance to `EqualityComparable` trait.

- `DynamicVector` has been renamed to `List`.  It has also moved from the `collections.vector`
  module to `collections.list` module.

- `StaticTuple` parameter order has changed to `StaticTuple[type, size]` for
  consistency with `SIMD` and similar collection types.

- The signature of the elementwise function has been changed. The new order is
  is `function`, `simd_width`, and then `rank`. As a result, the rank parameter
  can now be inferred and one can call elementwise via:

  ```mojo
  elementwise[func, simd_width](shape)
  ```

- For the time being, dynamic type value will be disabled in the language, e.g.
  the following will now fail with an error:

  ```mojo
  var t = Int  # dynamic type values not allowed

  struct SomeType: ...

  takes_type(SomeType)  # dynamic type values not allowed
  ```

  We want to take a step back and (re)design type valued variables,
  existentials, and other dynamic features for more 🔥. This does not affect
  type valued parameters, so the following will work as before:

  ```mojo
  alias t = Int  # still 🔥

  struct SomeType: ...

  takes_type[SomeType]()  # already 🔥
  ```

- `PythonObject` is now register-passable.

- `PythonObject.__iter__` now works correctly on more types of iterable Python
  objects.  Attempting to iterate over non-iterable objects will now raise an
  exception instead of behaving as if iterating over an empty sequence.
  `__iter__` also now borrows `self` rather than requiring `inout`, allowing
  code like `for value in my_dict.values():`.

- `List.push_back` has been removed.  Please use the `append` function instead.

- We took the opportunity to rehome some modules into their correct package
  as we were going through the process of open-sourcing the Mojo Standard
  Library.  Specifically, the following are some breaking changes worth
  calling out.  Please update your import statements accordingly.
  - `utils.list` has moved to `buffer.list`.
  - `rand` and `randn` functions in the `random` package that return a `Tensor`
     have moved to the `tensor` package.
  - `Buffer`, `NDBuffer`, and friends have moved from the `memory` package
     into a new `buffer` package.
  - The `trap` function has been renamed to `abort`.  It also has moved from the
    `debug` module to the `os` module.
  - `parallel_memcpy` has moved from the `memory` package into
     the `buffer` package.

- The `*_` expression in parameter expressions is now required to occur at the
  end of a positional parameter list, instead of being allowed in the middle.
  This is no longer supported: `SomeStruct[*_, 42]` but `SomeStruct[42, *_]` is
  still allowed. We narrowed this because we want to encourage type designers
  to get the order of parameters right, and want to extend `*_` to support
  keyword parameters as well in the future.

### ❌ Removed

- `let` declarations now produce a compile time error instead of a warning,
  our next step in [removing let
  declarations](https://github.com/modularml/mojo/blob/main/proposals/remove-let-decls.md).
  The compiler still recognizes the `let` keyword for now in order to produce
  a good error message, but that will be removed in subsequent releases.

- The `__get_address_as_lvalue` magic function has been removed.  You can now
  get an LValue from a `Pointer` or `Reference` by using the `ptr[]` operator.

- The type parameter for the `memcpy` function is now automatically inferred.
  This means that calls to `memcpy` of the form `memcpy[Dtype.xyz](...)` would
  no longer work and the user would have to change the code to `memcpy(...)`.

- `print_no_newline` has been removed.  Please use `print(end="")` instead.

- `memcpy` on `Buffer` has been removed in favor of just overloads for `Pointer`
  and `DTypePointer`.

- The functions `max_or_inf`, `min_or_neginf` have been removed from
  `math.limit` and just inlined into their use in SIMD.

### 🛠️ Fixed

- [#1362](https://github.com/modularml/mojo/issues/1362) - Parameter inference
  now recursively matches function types.
- [#951](https://github.com/modularml/mojo/issues/951) - Functions that were
  both `async` and `@always_inline` incorrectly errored.
- [#1858](https://github.com/modularml/mojo/issues/1858) - Trait with parametric
  methods regression.
- [#1892](https://github.com/modularml/mojo/issues/1892) - Forbid unsupported
  decorators on traits.
- [#1735](https://github.com/modularml/mojo/issues/1735) - Trait-typed values
  are incorrectly considered equal.
- [#1909](https://github.com/modularml/mojo/issues/1909) - Crash due to nested
  import in unreachable block.
- [#1921](https://github.com/modularml/mojo/issues/1921) - Parser crashes
  binding Reference to lvalue with subtype lifetime.
- [#1945](https://github.com/modularml/mojo/issues/1945) - `Optional[T].or_else()`
  should return `T` instead of `Optional[T]`.
- [#1940](https://github.com/modularml/mojo/issues/1940) - Constrain `math.copysign`
  to floating point or integral types.
- [#1838](https://github.com/modularml/mojo/issues/1838) - Variadic `print`
  does not work when specifying `end=""`
- [#1826](https://github.com/modularml/mojo/issues/1826) - The `SIMD.reduce` methods
  correctly handle edge cases where `size_out >= size`.

## v24.1.1 (2024-03-18)

This release includes installer improvements and enhanced error reporting for
installation issues. Otherwise it is functionally identical to Mojo 24.1.

## v24.1 (2024-02-29)

### 🔥 Legendary

- Mojo is now bundled with [the MAX platform](/max)!

  As such, the Mojo package version now matches the MAX version, which follows
  a `YY.MAJOR.MINOR` version scheme. Because this is our first release in 2024,
  that makes this version `24.1`.

- Mojo debugging support is here! The Mojo VS Code extension includes debugger
  support. For details, see [Debugging](/mojo/tools/debugging) in the Mojo
  Manual.

### ⭐️ New

- We now have a [`Set`](/mojo/stdlib/collections/set.html#set) type in our
  collections! `Set` is backed by a `Dict`, so it has fast add, remove, and `in`
  checks, and requires member elements to conform to the `KeyElement` trait.

  ```mojo
  from collections import Set

  var set = Set[Int](1, 2, 3)
  print(len(set))  # 3
  set.add(4)

  for element in set:
      print(element[])

  set -= Set[Int](3, 4, 5)
  print(set == Set[Int](1, 2))  # True
  print(set | Set[Int](0, 1) == Set[Int](0, 1, 2))  # True
  let element = set.pop()
  print(len(set))  # 1
  ```

- Mojo now supports the `x in y` expression as syntax sugar for
  `y.__contains__(x)` as well as `x not in y`.

- Mojo now has support for keyword-only arguments and parameters. For example:

  ```mojo
  fn my_product(a: Int, b: Int = 1, *, c: Int, d: Int = 2):
      print(a * b * c * d)

  my_product(3, c=5)     # prints '30'
  my_product(3, 5, d=7)  # error: missing 1 required keyword-only argument: 'c'
  ```

  This includes support for declaring signatures that use both variadic and
  keyword-only arguments/parameters. For example, the following is now possible:

  ```mojo
  fn prod_with_offset(*args: Int, offset: Int = 0) -> Int:
    var res = 1
    for i in range(len(args)):
        res *= args[i]
    return res + offset

  print(prod_with_offset(2, 3, 4, 10))         # prints 240
  print(prod_with_offset(2, 3, 4, offset=10))  # prints 34
  ```

  Note that variadic keyword-only arguments/parameters (for example, `**kwargs`)
  are not supported yet. That is, the following is not allowed:

  ```mojo
  fn variadic_kw_only(a: Int, **kwargs): ...
  ```

  For more information, see
  [Positional-only and keyword-only arguments](/mojo/manual/functions#positional-only-and-keyword-only-arguments)
  in the Mojo Manual.

- The `print()` function now accepts a keyword-only argument for the `end`
  which is useful for controlling whether a newline is printed or not
  after printing the elements.  By default, `end` defaults to "\n" as before.

- The Mojo SDK can now be installed on AWS Graviton instances.

- A new version of the [Mojo Playground](/mojo/playground) is available. The new
  playground is a simple interactive editor for Mojo code, similar to the Rust
  Playground or Go Playground. The old
  [JupyterLab based playground](https://playground.modular.com) will remain
  online until March 20th.

- The Mojo LSP server will now generate fixits for populating empty
  documentation strings:

  ```mojo
  fn foo(arg: Int):
    """""" # Unexpected empty documentation string
  ```

  Applying the fixit from above will generate:

  ```mojo
  fn foo(arg: Int):
    """[summary].

    Args:
        arg: [description].
    """
  ```

- Added new `*_` syntax that allows users to explicitly unbind any number of
  positional parameters. For example:

  ```mojo
  struct StructWithDefault[a: Int, b: Int, c: Int = 8, d: Int = 9]: pass

  alias all_unbound = StructWithDefault[*_]
  # equivalent to
  alias all_unbound = StructWithDefault[_, _, _, _]

  alias first_bound = StructWithDefault[5, *_]
  # equivalent to
  alias first_bound = StructWithDefault[5, _, _, _]

  alias last_bound = StructWithDefault[*_, 6]
  # equivalent to
  alias last_bound = StructWithDefault[_, _, _, 6]

  alias mid_unbound = StructWithDefault[3, *_, 4]
  # equivalent to
  alias mid_unbound = StructWithDefault[3, _, _, 4]
  ```

  As demonstrated above, this syntax can be used to explicitly unbind an
  arbitrary number of parameters, at the beginning, at the end, or in the
  middle of the operand list. Since these unbound parameters must be explicitly
  specified at some point, default values for these parameters are not applied.
  For example:

  ```mojo
  alias last_bound = StructWithDefault[*_, 6]
  # When using last_bound, you must specify a, b, and c. last_bound
  # doesn't have a default value for `c`.
  var s = last_bound[1, 2, 3]()
  ```

  For more information see the Mojo Manual sections on
  [partially-bound types](/mojo/manual/parameters/#fully-bound-partially-bound-and-unbound-types)
  and
  [automatic parameterization of functions](/mojo/manual/parameters/#automatic-parameterization-of-functions).

- [`DynamicVector`](/mojo/stdlib/collections/vector.html#dynamicvector) now
  supports iteration. Iteration values are instances of
  [Reference](/mojo/stdlib/memory/unsafe#reference) and require dereferencing:

  ```mojo
  var v: DynamicVector[String]()
  v.append("Alice")
  v.append("Bob")
  v.append("Charlie")
  for x in v:
    x[] = str("Hello, ") + x[]
  for x in v:
    print(x[])
  ```

- `DynamicVector` now has
  [`reverse()`](/mojo/stdlib/collections/vector.html#reverse) and
  [`extend()`](/mojo/stdlib/collections/vector.html#extend) methods.

- The `mojo package` command now produces compilation agnostic packages.
  Compilation options such as O0, or --debug-level, are no longer needed or
  accepted. As a result, packages are now smaller, and extremely portable.

- Initializers for `@register_passable` values can (and should!) now be
  specified with `inout self` arguments just like memory-only types:

  ```mojo
  @register_passable
  struct YourPair:
    var a: Int
    var b: Int
    fn __init__(inout self):
        self.a = 42
        self.b = 17
    fn __copyinit__(inout self, existing: Self):
        self.a = existing.a
        self.b = existing.b
  ```

  This form makes the language more consistent, more similar to Python, and
  easier to implement advanced features for.  There is also no performance
  impact of using this new form: the compiler arranges to automatically return
  the value in a register without requiring you to worry about it.

  The older `-> Self:` syntax is still supported in this release, but will be
  removed in a subsequent one, so please migrate your code.  One thing to watch
  out for: a given struct should use one style or the other, mixing some of
  each won't work well.

- The standard library `slice` type has been renamed to
  [`Slice`](/mojo/stdlib/builtin/builtin_slice#slice), and a `slice`
  function has been introduced.  This makes Mojo closer to Python and makes the
  `Slice` type follow the naming conventions of other types like `Int`.

- "Slice" syntax in subscripts is no longer hard coded to the builtin `slice`
  type: it now works with any type accepted by a container's `__getitem__()`
  method. For example:

  ```mojo
  @value
  struct UnusualSlice:
    var a: Int
    var b: Float64
    var c: String

  struct YourContainer:
    fn __getitem__(self, slice: UnusualSlice) -> T: ...
  ```

  Given this implementation, you can subscript into an instance of
  `YourContainer` like `yc[42:3.14:"🔥"]` and the three values are passed to the
  `UnusualSlice` constructor.

- The `__refitem__()` accessor method may now return a
  [`Reference`](/mojo/stdlib/memory/unsafe#reference) instead of having to
  return an MLIR internal reference type.

- Added [`AnyPointer.move_into()`](/mojo/stdlib/memory/anypointer.html#move_into)
  method, for moving a value from one pointer memory location to another.

- Added built-in [`hex()`](/mojo/stdlib/builtin/hex#hex) function, which can be
  used to format any value whose type implements the
  [`Intable`](/mojo/stdlib/builtin/int#intable) trait as a hexadecimal string.

- [`PythonObject`](/mojo/stdlib/python/object#pythonobject) now implements
  `__is__` and `__isnot__` so that you can use expressions of the form `x is y`
  and `x is not y` with `PythonObject`.

- [`PythonObject`](/mojo/stdlib/python/object#pythonobject) now conforms to the
  `SizedRaising` trait. This means the built-in
  [`len()`](/mojo/stdlib/builtin/len#len) function now works on `PythonObject`.

- The `os` package now contains the [`stat()`](/mojo/stdlib/os/fstat#stat)
  and [`lstat()`](/mojo/stdlib/os/fstat#lstat) functions.

- A new [`os.path`](/mojo/stdlib/os/path/path) package now allows you to query
  properties on paths.

- The `os` package now has a
  [`PathLike`](/mojo/stdlib/os/pathlike.html#pathlike) trait. A struct conforms
  to the `PathLike` trait by implementing the `__fspath__()` function.

- The [`pathlib.Path`](/mojo/stdlib/pathlib/path#path) now has functions to
  query properties of the path.

- The [`listdir()`](/mojo/stdlib/pathlib/path#listdir) method now exists on
  [`pathlib.Path`](/mojo/stdlib/pathlib/path) and also exists in the `os`
  module to work on `PathLike` structs. For example, the following sample
  lists all the directories in the `/tmp` directory:

  ```mojo
  from pathlib import Path

  fn walktree(top: Path, inout files: DynamicVector[Path]):
      try:
          var ls = top.listdir()
          for i in range(len(ls)):
              var child = top / ls[i]
              if child.is_dir():
                  walktree(child, files)
              elif child.is_file():
                  files.append(child)
              else:
                  print("Skipping '" + str(child) + "'")
      except:
          return

  fn main():
      var files = DynamicVector[Path]()

      walktree(Path("/tmp"), files)

      for i in range(len(files)):
          print(files[i])
  ```

- The [`find()`](/mojo/stdlib/builtin/string_literal#find),
  [`rfind()`](/mojo/stdlib/builtin/string_literal#rfind),
  [`count()`](/mojo/stdlib/builtin/string_literal#count), and
  [`__contains__()`](/mojo/stdlib/builtin/string_literal#__contains__) methods
  now work on string literals. This means that you can write:

  ```mojo
  if "Mojo" in "Hello Mojo":
    ...
  ```

- Breakpoints can now be inserted programmatically within the code using the
  builtin [`breakpoint()`](/mojo/stdlib/builtin/breakpoint#breakpoint) function.

  Note: on Graviton instances, the debugger might not be able to resume after
  hitting this kind of breakpoint.

- Added a builtin [`Boolable`](/mojo/stdlib/builtin/bool#boolable) trait that
  describes a type that can be represented as a boolean value. To conform to the
  trait, a type must implement the `__bool__()` method.

- Modules within packages can now use purely relative `from` imports:

  ```mojo
  from . import another_module
  ```

- Trivial types, like MLIR types and function types, can now be bound implicitly
  to traits that require copy constructors or move constructors, such as
  [`Movable`](/mojo/stdlib/builtin/value.html#movable),
  [`Copyable`](/mojo/stdlib/builtin/value.html#copyable), and
  [`CollectionElement`](/mojo/stdlib/builtin/value#collectionelement).

- A new magic `__lifetime_of(expr)` call will yield the lifetime of a memory
  value.  We hope and expect that this will eventually be replaced by
  `Reference(expr).lifetime` as the parameter system evolves, but this is
  important in the meantime for use in function signatures.

- A new magic `__type_of(expr)` call will yield the type of a value. This allows
  one to refer to types of other variables. For example:

  ```mojo
  fn my_function(x: Int, y: __type_of(x)) -> Int:
    let z : __type_of(x) = y
    return z
  ```

### 🦋 Changed

- As another step towards [removing let
  declarations](https://github.com/modularml/mojo/blob/main/proposals/remove-let-decls.md)
  we have removed support for let declarations inside the compiler.  To ease
  migration, we parse `let` declarations as a `var` declaration so your code
  won't break.  We emit a warning about this, but please switch your code to
  using `var` explicitly, because this migration support will be removed in a
  subsequent update.

  ```mojo
  fn test():
    # treated as a var, but please update your code!
    let x = 42  # warning: 'let' is being removed, please use 'var' instead
    x = 9
  ```

- It is no longer possible to explicitly specify implicit argument parameters in
  [automatically parameterized
  functions](/mojo/manual/parameters/#automatic-parameterization-of-functions)).
  This ability was an oversight and this is now an error:

  ```mojo
  fn autoparameterized(x: SIMD):
      pass

  autoparameterized[DType.int32, 1](3) # error: too many parameters
  ```

- `vectorize_unroll` has been removed, and
  [`vectorize`](/mojo/stdlib/algorithm/functional#vectorize) now has a parameter
  named `unroll_factor` with a default value of 1. Increasing `unroll_factor`
  may improve performance at the cost of binary size. See the
  [loop unrolling blog here](https://www.modular.com/blog/what-is-loop-unrolling-how-you-can-speed-up-mojo)
  for more details.

- The `vectorize` signatures have changed with the closure `func` moved to the
  first parameter:

  ```mojo
  vectorize[func, width, unroll_factor = 1](size)
  vectorize[func, width, size, unroll_factor = 1]()
  ```

  The doc string has been updated with examples demonstrating the difference
  between the two signatures.

- The `unroll` signatures have changed with the closure `func` moved to the
  first parameter:

  ```mojo
  unroll[func, unroll_count]()
  ```

- The signature of the [`NDBuffer`](/mojo/stdlib/memory/buffer#ndbuffer) and
  [`Buffer`](/mojo/stdlib/memory/buffer#buffer) types have changed. Now, both
  take the type as the first parameter and no longer require the shape
  parameter. This allows you to use these types and have sensible defaults.
  For example:

  ```mojo
  NDBuffer[DType.float32, 3]
  ```

  is equivalent to

  ```mojo
  NDBuffer[DType.float32, 3, DimList.create_unknown[3]()]
  ```

  Users can still specify the static shape (if known) to the type:

  ```mojo
  NDBuffer[DType.float32, 3, DimList(128, 128, 3)]
  ```

- The error message for missing function arguments is improved: instead of
  describing the number of arguments (e.g. `callee expects at least 3 arguments,
  but 1 was specified`) the missing arguments are now described by
  name (e.g. `missing 2 required positional arguments: 'b', 'c'`).

- The [`CollectionElement`](/mojo/stdlib/builtin/value#collectionelement) trait
  is now a built-in trait and has been removed from `collections.vector`.

- The `DynamicVector(capacity: Int)` constructor has been changed to take
  `capacity` as a keyword-only argument to prevent implicit conversion from
  `Int`.

- [`Variant.get[T]()`](/mojo/stdlib/utils/variant#get) now returns a
  [`Reference`](/mojo/stdlib/memory/unsafe#reference) to the value rather than a
  copy.

- The [`String`](/mojo/stdlib/builtin/string.html#string) methods `tolower()`
  and `toupper()` have been renamed to `str.lower()` and `str.upper()`.

- The `ref` and `mutref` identifiers are no longer reserved as Mojo keywords.
  We originally thought about using those as language sugar for references, but
  we believe that generic language features combined with the
  [`Reference`](/mojo/stdlib/memory/unsafe#reference) type will provide a good
  experience without dedicated sugar.

### 🛠️ Fixed

- [#435](https://github.com/modularml/mojo/issues/435)
  Structs with Self type don't always work.
- [#1540](https://github.com/modularml/mojo/issues/1540)
  Crash in register_passable self referencing struct.
- [#1664](https://github.com/modularml/mojo/issues/1664) - Improve error
  message when `StaticTuple` is constructed with a negative size for
  the number of elements.
- [#1679](https://github.com/modularml/mojo/issues/1679) - crash on SIMD of zero
  elements.
- Various crashes on invalid code:
  [#1230](https://github.com/modularml/mojo/issues/1230),
  [#1699](https://github.com/modularml/mojo/issues/1699),
  [#1708](https://github.com/modularml/mojo/issues/1708)
- [#1223](https://github.com/modularml/mojo/issues/1223) - Crash when parametric
  function is passed as (runtime) argument. The parser now errors out instead.
- [#1530](https://github.com/modularml/mojo/issues/1530) - Crash during
  diagnostic emission for parameter deduction failure.
- [#1538](https://github.com/modularml/mojo/issues/1538) and [#1607](
  https://github.com/modularml/mojo/issues/1607) - Crash when returning type
  value instead of instance of expected type. This is a common mistake and the
  error now includes a hint to point users to the problem.
- [#1613](https://github.com/modularml/mojo/issues/1613) - Wrong type name in
  error for incorrect `self` argument type in trait method declaration.
- [#1670](https://github.com/modularml/mojo/issues/1670) - Crash on implicit
  conversion in a global variable declaration.
- [#1741](https://github.com/modularml/mojo/issues/1741) - Mojo documentation
  generation doesn't show `inout`/`owned` on variadic arguments.
- [#1621](https://github.com/modularml/mojo/issues/1621) - VS Code does not
  highlight `raises` and `capturing` in functional type expressions.
- [#1617](https://github.com/modularml/mojo/issues/1617) - VS Code does not
  highlight `fn` in specific contexts.
- [#1740](https://github.com/modularml/mojo/issues/1740) - LSP shows unrelated
  info when hovering over a struct.
- [#1238](https://github.com/modularml/mojo/issues/1238) - File shadows Mojo
  package path.
- [#1429](https://github.com/modularml/mojo/issues/1429) - Crash when using
  nested import statement.
- [#1322](https://github.com/modularml/mojo/issues/1322) - Crash when missing
  types in variadic argument.
- [#1314](https://github.com/modularml/mojo/issues/1314) - Typecheck error when
  binding alias to parametric function with default argument.
- [#1248](https://github.com/modularml/mojo/issues/1248) - Crash when importing
  from file the same name as another file in the search path.
- [#1354](https://github.com/modularml/mojo/issues/1354) - Crash when importing
  from local package.
- [#1488](https://github.com/modularml/mojo/issues/1488) - Crash when setting
  generic element field.
- [#1476](https://github.com/modularml/mojo/issues/1476) - Crash in interpreter
  when calling functions in parameter context.
- [#1537](https://github.com/modularml/mojo/issues/1537) - Crash when copying
  parameter value.
- [#1546](https://github.com/modularml/mojo/issues/1546) - Modify nested vector
  element crashes parser.
- [#1558](https://github.com/modularml/mojo/issues/1558) - Invalid import causes
  parser to crash.
- [#1562](https://github.com/modularml/mojo/issues/1562) - Crash when calling
  parametric type member function.
- [#1577](https://github.com/modularml/mojo/issues/1577) - Crash when using
  unresolved package as a variable.
- [#1579](https://github.com/modularml/mojo/issues/1579) - Member access into
  type instances causes a crash.
- [#1602](https://github.com/modularml/mojo/issues/1602) - Interpreter failure
  when constructing strings at compile time.
- [#1696](https://github.com/modularml/mojo/issues/1696) - Fixed an issue that
  caused syntax highlighting to occasionally fail.
- [#1549](https://github.com/modularml/mojo/issues/1549) - Fixed an issue when
  the shift amount is out of range in `SIMD.shift_left` and `SIMD.shift_right`.

## v0.7.0 (2024-01-25)

### ⭐️ New

- A new Mojo-native dictionary type,
  [`Dict`](/mojo/stdlib/collections/dict.html) for storing key-value pairs.
  `Dict` stores values that conform to the
  [`CollectionElement`](/mojo/stdlib/builtin/value#collectionelement)
  trait. Keys need to conform to the new
  [`KeyElement`](/mojo/stdlib/collections/dict.html#keyelement) trait, which is
  not yet implemented by other standard library types. In the short term, you
  can create your own wrapper types to use as keys. For example, the following
  sample defines a `StringKey` type and uses it to create a dictionary that maps
  strings to `Int` values:

  ```mojo
  from collections.dict import Dict, KeyElement

  @value
  struct StringKey(KeyElement):
      var s: String

      fn __init__(inout self, owned s: String):
          self.s = s ^

      fn __init__(inout self, s: StringLiteral):
          self.s = String(s)

      fn __hash__(self) -> Int:
          return hash(self.s)

      fn __eq__(self, other: Self) -> Bool:
          return self.s == other.s

  fn main() raises:
      var d = Dict[StringKey, Int]()
      d["cats"] = 1
      d["dogs"] = 2
      print(len(d))         # prints 2
      print(d["cats"])      # prints 1
      print(d.pop("dogs"))  # prints 2
      print(len(d))         # prints 1
  ```

  We plan to add `KeyElement` conformance to standard library types in
  subsequent releases.

- Users can opt-in to assertions used in the standard library code by
  specifying `-D MOJO_ENABLE_ASSERTIONS` when invoking `mojo` to
  compile your source file(s).  In the case that an assertion is fired,
  the assertion message will be printed along with the stack trace
  before the program exits.  By default, assertions are _not enabled_
  in the standard library right now for performance reasons.

- The Mojo Language Server now implements the References request. IDEs use
  this to provide support for **Go to References** and **Find All References**.
  A current limitation is that references outside of the current document are
  not supported, which will be addressed in the future.

- The [`sys.info`](/mojo/stdlib/sys/info) module now includes
  `num_physical_cores()`, `num_logical_cores()`, and `num_performance_cores()`
  functions.

- Homogenous variadic arguments consisting of memory-only types, such as
  `String` are more powerful and easier to use. These arguments are projected
  into a
  [`VariadicListMem`](/mojo/stdlib/builtin/builtin_list.html#variadiclistmem).

  (Previous releases made it easier to use variadic lists of register-passable
  types, like `Int`.)

  Subscripting into a `VariadicListMem` now returns the element instead of an
  obscure internal type. In addition, we now support `inout` and `owned`
  variadic arguments:

  ```mojo
  fn make_worldly(inout *strs: String):
      # This "just works" as you'd expect!
      for i in range(len(strs)):
          strs[i] += " world"
  fn main():
      var s1: String = "hello"
      var s2: String = "konnichiwa"
      var s3: String = "bonjour"
      make_worldly(s1, s2, s3)
      print(s1)  # hello world
      print(s2)  # konnichiwa world
      print(s3)  # bonjour world
  ```

  (Previous releases made it easier to use variadic lists, but subscripting into
  a `VariadicListMem` returned a low-level pointer, which required the user to
  call `__get_address_as_lvalue()` to access the element.)

  Note that subscripting the variadic list works nicely as above, but
  iterating over the variadic list directly with a `for` loop produces a
  [`Reference`](/mojo/stdlib/memory/unsafe#reference) (described below) instead
  of the desired value, so an extra subscript is required; We intend to fix this
  in the future.

  ```mojo
  fn make_worldly(inout *strs: String):
      # Requires extra [] to dereference the reference for now.
      for i in strs:
          i[] += " world"
  ```

  Heterogenous variadic arguments have not yet been moved to the new model, but
  will in future updates.

  Note that for variadic arguments of register-passable types like `Int`, the
  variadic list contains values, not references, so the dereference operator
  (`[]`) is not required. This code continues to work as it did previously:

  ```mojo
  fn print_ints(*nums: Int):
      for num in nums:
          print(num)
      print(len(nums))
  ```

- Mojo now has a prototype version of a safe
  [`Reference`](/mojo/stdlib/memory/unsafe#reference) type. The compiler's
  lifetime tracking pass can reason about references to safely extend local
  variable lifetime, and check indirect access safety.  The `Reference` type
  is brand new (and currently has no syntactic sugar) so it must be explicitly
  dereferenced with an empty subscript: `ref[]` provides access to the
  underlying value.

  ```mojo
  fn main():
      var a : String = "hello"
      var b : String = " references"

      var aref = Reference(a)
      aref[] += b
      print(a)  # prints "hello references"

      aref[] += b
      # ^last use of b, it is destroyed here.

      print(aref[]) # prints "hello references references"
      # ^last use of a, it is destroyed here.
  ```

  While the `Reference` type has the same in-memory representation as a C
  pointer or the Mojo `Pointer` type, it also tracks a symbolic "lifetime" value
  so the compiler can reason about the potentially accessed set of values.  This
  lifetime is part of the static type of the reference, so it propagates through
  generic algorithms and abstractions built around it.

  The `Reference` type can form references to both mutable and immutable memory
  objects, e.g. those on the stack or borrowed/inout/owned function arguments.
  It is fully parametric over mutability, eliminating the [problems with code
  duplication due to mutability
  specifiers](https://duckki.github.io/2024/01/01/inferred-mutability.html) and
  provides the base for unified user-level types. For example, it could be
  used to implement an array slice object that handles both mutable and immutable
  array slices.

  While this is a major step forward for the lifetimes system in Mojo, it is
  still _very_ early and awkward to use.  Notably, there is no syntactic sugar
  for using references, such as automatic dereferencing. Several aspects of it
  need to be more baked. It is getting exercised by variadic memory arguments,
  which is why they are starting to behave better now.

  Note: the safe `Reference` type and the unsafe pointer types are defined in
  the same module, currently named `memory.unsafe`. We expect to restructure
  this module in a future release.

- Mojo now allows types to implement `__refattr__()` and `__refitem__()` to
  enable attribute and subscript syntax with computed accessors that return
  references. For common situations where these address a value in memory this
  provides a more convenient and significantly more performant alternative to
  implementing the traditional get/set pairs.  Note: this may be changed in the
  future when references auto-dereference—at that point we may switch to just
  returning a reference from `__getattr__()`.
- Parametric closures can now capture register passable typed values by copy
  using the `__copy_capture` decorator. For example, the following code will
  print `5`, not `2`.

  ```mojo
  fn foo(x: Int):
      var z = x

      @__copy_capture(z)
      @parameter
      fn print_elt[T: Stringable](a: T):
          print_string(" ")
          print_string(a)
      rest.each[print_elt]()
  ```

- The `sys` module now contains an `exit` function that would exit a Mojo
  program with the specified error code.

- The constructors for `tensor.Tensor` have been changed to be more consistent.
  As a result, one has to pass in the shape as first argument (instead of the
  second) when constructing a tensor with pointer data.

- The constructor for `tensor.Tensor` will now splat a scalar if its passed in.
  For example, `Tensor[DType.float32](TensorShape(2,2), 0)` will construct a
  `2x2` tensor which is initialized with all zeros. This provides an easy way
  to fill the data of a tensor.

- The `mojo build` and `mojo run` commands now support a `-g` option. This
  shorter alias is equivalent to writing `--debug-level full`. This option is
  also available in the `mojo debug` command, but is already the default.

- `PythonObject` now conforms to the `KeyElement` trait, meaning that it can be
  used as key type for `Dict`. This allows on to easily build and interact with
  Python dictionaries in mojo:

  ```mojo
  def main():
      d = PythonObject(Dict[PythonObject, PythonObject]())
      d["foo"] = 12
      d[7] = "bar"
      d["foo"] = [1, 2, "something else"]
      print(d)  # prints `{'foo': [1, 2, 'something else'], 7: 'bar'}`
  ```

- `List` now has a `pop(index)` API for removing an element
  at a particular index.  By default, `List.pop()` removes the last element
  in the list.

- `Dict` now has a `update()` method to update keys/values from another `Dict`.

- A low-level `__get_mvalue_as_litref(x)` builtin was added to give access to
  the underlying memory representation as a `!lit.ref` value without checking
  initialization status of the underlying value.  This is useful in very
  low-level logic but isn't designed for general usability and will likely
  change in the future.

- The `testing.assert_almost_equal` and `math.isclose` functions now have an
  `equal_nan` flag. When set to True, then NaNs are considered equal.

### 🦋 Changed

- The behavior of `mojo build` when invoked without an output `-o` argument has
  changed slightly: `mojo build ./test-dir/program.mojo` now outputs an
  executable to the path `./program`, whereas before it would output to the path
  `./test-dir/program`.
- The REPL no longer allows type level variable declarations to be
  uninitialized, e.g. it will reject `var s: String`.  This is because it does
  not do proper lifetime tracking (yet!) across cells, and so such code would
  lead to a crash.  You can work around this by initializing to a dummy value
  and overwriting later.  This limitation only applies to top level variables,
  variables in functions work as they always have.
- `AnyPointer` got renamed to `UnsafePointer` and is now Mojo's preferred unsafe
  pointer type.  It has several enhancements, including:
  1) The element type can now be `AnyType`: it doesn't require `Movable`.
  2) Because of this, the `take_value`, `emplace_value`, and `move_into` methods
     have been changed to be top-level functions, and were renamed to
     `move_from_pointee`, `initialize_pointee` and `move_pointee` respectively.
  3) A new `destroy_pointee` function runs the destructor on the pointee.
  4) `UnsafePointer` can be initialized directly from a `Reference` with
     `UnsafePointer(someRef)` and can convert to an immortal reference with
     `yourPointer[]`.  Both infer element type and address space.
- All of the pointers got a pass of cleanup to make them more consistent, for
  example the `unsafe.bitcast` global function is now a consistent `bitcast`
  method on the pointers, which can convert element type and address space.
- The `Reference` type has several changes, including:
  1) It is now located in `memory.reference` instead of `memory.unsafe`.
  2) `Reference` now has an unsafe `unsafe_bitcast` method like `UnsafePointer`.
  3) Several unsafe methods were removed, including `offset`,
     `destroy_element_unsafe` and `emplace_ref_unsafe`. This is because
     `Reference` is a safe type - use `UnsafePointer` to do unsafe operations.

- The `mojo package` command no longer supports the `-D` flag. All compilation
  environment flags should be provided at the point of package use
  (e.g. `mojo run` or `mojo build`).

- `parallel_memcpy` function has moved from the `buffer` package to the `algorithm`
  package.  Please update your imports accordingly.

### ❌ Removed

- Support for "register only" variadic packs has been removed. Instead of
  `AnyRegType`, please upgrade your code to `AnyType` in examples like this:

  ```mojo
  fn your_function[*Types: AnyRegType](*args: *Ts): ...
  ```

  This move gives you access to nicer API and has the benefit of being memory
  safe and correct for non-trivial types.  If you need specific APIs on the
  types, please use the correct trait bound instead of `AnyType`.

- `List.pop_back()` has been removed.  Use `List.pop()` instead which defaults
  to popping the last element in the list.

- `SIMD.to_int(value)` has been removed.  Use `int(value)` instead.

- The `__get_lvalue_as_address(x)` magic function has been removed.  To get a
  reference to a value use `Reference(x)` and if you need an unsafe pointer, you
  can use `UnsafePointer.address_of(x)`.

### 🛠️ Fixed

- [#516](https://github.com/modularml/mojo/issues/516) and
  [#1817](https://github.com/modularml/mojo/issues/1817) and many others, e.g.
  "Can't create a function that returns two strings"

- [#1178](https://github.com/modularml/mojo/issues/1178) (os/kern) failure (5)

- [#1609](https://github.com/modularml/mojo/issues/1609) alias with
  `DynamicVector[Tuple[Int]]` fails.

- [#1987](https://github.com/modularml/mojo/issues/1987) Defining `main`
  in a Mojo package is an error, for now. This is not intended to work yet,
  erroring for now will help to prevent accidental undefined behavior.

- [#1215](https://github.com/modularml/mojo/issues/1215) and
  [#1949](https://github.com/modularml/mojo/issues/1949) The Mojo LSP server no
  longer cuts off hover previews for functions with functional arguments,
  parameters, or results.

- [#1245](https://github.com/modularml/mojo/issues/1245) [Feature Request]
  Parameter Inference from Other Parameters.

- [#1901](https://github.com/modularml/mojo/issues/1901) Fixed Mojo LSP and
  documentation generation handling of inout arguments.

- [#1913](https://github.com/modularml/mojo/issues/1913) - `0__` no longer
  crashes the Mojo parser.

- [#1924](https://github.com/modularml/mojo/issues/1924) JIT debugging on Mac
  has been fixed.

- [#1941](https://github.com/modularml/mojo/issues/1941) Mojo variadics don't
  work with non-trivial register-only types.

- [#1963](https://github.com/modularml/mojo/issues/1963) `a!=0` is now parsed
  and formatted correctly by `mojo format`.

- [#1676](https://github.com/modularml/mojo/issues/1676) Fix a crash related to
  `@value` decorator and structs with empty body.

- [#1917](https://github.com/modularml/mojo/issues/1917) Fix a crash after
  syntax error during tuple creation

- [#2006](https://github.com/modularml/mojo/issues/2006) The Mojo LSP now
  properly supports signature types with named arguments and parameters.

- [#2007](https://github.com/modularml/mojo/issues/2007) and
  [#1997](https://github.com/modularml/mojo/issues/1997) The Mojo LSP no longer
  crashes on certain types of closures.

- [#1675](https://github.com/modularml/mojo/issues/1675) Ensure `@value`
  decorator fails gracefully after duplicate field error.
