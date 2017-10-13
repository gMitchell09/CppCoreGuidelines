Rules:
1. If arguments are not PODs then prefer passing by const reference
2. If function is a member function prefer making it a const function if it doesn't mutate the state of the class.
3. "Say what should be done rather than just how it should be done"

class Foo {
public:
  bool Bar(int i, const Baz& baz) const {
    return i == baz.bleh(this->m_thing);
  }
private:
  Blah m_b;
}

Immediate Rejection Criteria:
Resource leaks
Range errors
Unions
Repeated code (where n >= 4)
Global Variables
Singletons
void*

Red Flags:
Raw pointers
New/Delete
Malloc/Free
non-POD arguments not being passed by const reference
Member functions that don't mutate state not being declared const
C-Style Casts: (uint32_t)value
"What is not checkable at compile-time should be checkable at runtime": https://github.com/gMitchell09/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#p6-what-cannot-be-checked-at-compile-time-should-be-checkable-at-run-time
Repeated code (where n == 3)

Yellow Flags:
Some unnecessary casts?????
Usage of index-based loops instead of for-in style loops.
f(T*, int) argument types vs std::containers or std::span
Repeated code (where n < 3)
std::map::operator[] should be avoided
