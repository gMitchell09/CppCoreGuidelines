# C++ Rules

[TOC]

## High Level Rules
1. Prefer immutable data over mutable data (const const const)
1. Prefer references over copies
1. "Say what should be done rather than just how it should be done"
	1. Does a member function leave the instance in the same state that it starts?  Mark it as const.
    1. Does a member not throw an exception?  Mark it as noexcept.

```Cpp
class Foo {
public:
  bool Bar(int i, const Baz& baz) const {
    return i == baz.bleh(this->m_thing);
  }

  void Baz(const Blah& b) { // No const because mutates
  	m_b = b;
  }
private:
  Blah m_b;
}
```

## How to use this guide
This guide is intended for providing objective rules to use during the initial stages of a code review.  Code that contains anything in the [Black Flags]() section must be... well... immediately rejected.  These are our strict "hard-rules" that ensure that our code is safe and maintainable.

The [Red Flags]() section represents rules that even if not strictly followed *could* result in passable code if thoroughly checked.  Each red flag should add about 1 hour to the review time to verify correctness.

The [Yellow Flags]() section represents rules that *should* result in safe code but could be written better and safer but are fine to be used in long-term production code with proper verification.  Each yellow flag should add about 1/2 hour to review time to verify correctness.

## Black Flags
- Provable resource leaks
- Range errors
- Unions
- Repeated code (where n >= 4)
- Global Variables
- Singletons
- void*
- `using namespace *` in a header
	- Long namespaces can be abbreviated with:
	```CPP
		namespace tcp = boost::asio::tcp;
    ```
- Using "unsafe" data for any sort of memory operations (indexing into array, malloc, dereferencing pointers).  Unsafe data must be made safe with bounds checking.
- Do not use '\n' for a newline.  Use std::endl.
- Usage of compiler-specific extensions
- Usage of platform-specific libraries
- Usage of NULL or even worse, 0, for checking pointers.  USE nullptr.
- Re-creation of functions that exist in STL (e.g.):
	- std::min, std::max, std::minmax
	- std::find(_if(_not))
	- std::find_first_of
	- std::for_each
	- std::(all|any|none)_of
	- std::count(_if)
	- std::copy(_if|_n)
    - std::transform
    - Just familiarize yourself and start using #include <algorithm>'s

## Red Flags
- Raw pointers:
	- Upgraded to [Black Flag]() if 
    	- Use breaks [Strict Aliasing]() rules
    	- [lifetime is questionable](Provable resource leaks)
- New/Delete
- Malloc/Free
- non-POD arguments not being passed by const reference
- Member functions that don't mutate state not being declared const
- C-Style Casts: (uint32_t)value
- "What is not checkable at compile-time should be checkable at runtime": https://github.com/gMitchell09/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#p6-what-cannot-be-checked-at-compile-time-should-be-checkable-at-run-time
- Repeated code (where n == 3)
- `dynamic_cast<>`
- Formatting inconsistency
- Unnecessary #includes when class forward-declarations could be used instead
- Missing #include's that are included by some other dependency.  We want the #include list to mark all dependencies of a module.
- Preprocessor Macros
- Usage of `enum X { ... }` instead of `enum class X { ... }`

## Yellow Flags
- Some unnecessary casts?????
- Usage of index-based loops instead of for-in style loops.
- `f(T*, int)` argument types vs std::containers or std::span
- Repeated code (where n < 3)
- std::map::operator[] should be avoided.  Corrolary: ALL maps should be const UNLESS the code must mutate them.
- Compilation warnings.  Let us set a goal to reduce compiler warnings to zero.
- Usage of str \=\= ""  or str.size() \=\= 0 instead of str.empty()

## Green Flags (these are good things that should give reviewers warm fuzzies)
- Use of boost::optional to return values from functions that can fail but where failure shouldn't halt the program
- ...

## Style Guide
Follows [Mozilla Style Guide](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Coding_Style) with exceptions as follows:
- Indentation: 4 spaces per logic level
- Curly brace placement is up to the developer AS LONG AS IT IS CONSISTENT
- Ignore argument variable name prefix (function arguments SHOULD NOT begin with a)
- Scoped member variables will have the following prefixes:
	- s_ for statically visible members
	- m_ for private members
	- p_ for protected members
- IGNORE "enum values start with e".  This is a holdover from when enums were scoped to the parent namespace.  Use `enum class` everywhere and this is unnecessary.
- Remove unnecessary control flow statements (e.g.):
```cpp
if (bad) {
	std::cout << "Whoopsies" << std::endl;
    return {};
} else {
	return myCoolThing;
}
```
Should be:
```cpp
if (bad) {
	std::cout << "Whoopsies" << std::endl;
    return {};
}
return myCoolThing;
```

- Include order is as follows:
	- \#include "Foo.h" // IF IN Foo.cpp
	- newline
	- STL includes
	- newline
	- boost includes
	- newline
	- QT includes
	- newline
	- \#include "local_file_includes"

## Review Rules:
- Changes to existing files must be reviewed by the original author of the file if different from editor.
- A review must __start__ within 1 day of request.  Must take a minimum
