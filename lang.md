Rough thoughts and ideas on the language I want to write for my senior thesis.

### Syntax

 + Possibly allow some kind of off-side rule syntax (not everyone likes S-expressions)
     + look into [Scheme RFI 49](http://srfi.schemers.org/srfi-49/srfi-49.html)'s "I-expressions"
     + and [T-expressions](http://srfi.schemers.org/srfi-110/srfi-110.html)
     + this doesn't have to be implemented for my comp, but it would be nice to provide eventually. It might make the syntax more friendly to systems programmers with experience in non-LISP languages.
     + parsing this should not be all that much harder than S-expressions.
 + Types/type syntax
     + type annotations of the `variable: Type` or `variable : Type` form
       + a là Scala/ML/Rust/etc
       + this is easier to parse than Java/C style
       + and just seems Morally Correct
     + S-expressions for lists, obviously (`(a b c)` and `(a . (b . c)) forms`)
     + square braces?
         + I like how Scheme allows matching square braces to be substituted for matching perens, this makes large nested constructs much more readable
         + Of course, a systems language needs an easy to use array syntax, too, and square braces for arrays is nearly universal...
         + Hmm.
     + Curlies
         + Once again, a systems language will likely need some kind of struct syntax for talking to C libraries, so curlies will probably be used for this.
         + It would be nice if these could compile directly to C structs.
         + Possibly steal ideas from Clojure's dict syntax?
         + T-expression style curly-infix is nice, though.

Example code snippets:

Lisp-style:
```lisp
(def fac (n: Int)
  (if (= n 1) 1
    (* n (fac (- n 1)))
  )
)
```
I-expression style with curly infix notation:
```lisp
def fac (n: Int) ; this looks _almost_ like Python!
  if {n = 1} 
    1
    {n * fac {n - 1}}
```
### Types

  + ADTs
      + LISPs generally don't have ADTs, but that's because the ADT is sort of inherently a strongly-typed language construct.
      + This is a strongly-typed language.
      + ADTs allow you to express things that are hard to express in strongly-typed languages
          + (look at Haskell's `data` and Rust's `enum`!)
          + These are powerful constructs and some ideas are hard to express without them (look at all the unnecessary Java code that could be avoided with ADTs)
      + ADTs, if implemented correctly, can be a wholly compile-time abstraction
          + this is huge; 0-cost abstractions are basically the theme of this language
          + necessary for systems work
          + however, if ADTs are just there for the compiler's benefit and never reified, that means that they can't be used for ABI code
              + this kind of binds us to including dependencies as source and compiling everything together
              + a là Cargo
              + if the compiler is Reasonably Fast, I think this is a _much_ better system anyway
  + typeclasses
    + honestly like, one of the greatest Haskell inventions
    + Rust's `trait` system is a good version of typeclasses for regular people (the keyword `trait` seems like a good way to not confuse OO people...)
    + typeclasses are nice since a lot of their overhead can be resolved at compile time
    + why would you not want this?
  + lists
      + it would be nice if these could be wholly stack-allocated
          + otherwise we'd need to write some kind of ref-counting pointer, and that would take work
          + also, I really don't want a core language construct (since this is LISP-oid) to necessitate GC, because the whole point is that it's a stack language with memory safety, and if you suddenly can't use lists that's crippling
          + unfortunately this might make it hard to have some common FP list advantages (immutable lists sharing tails, etc, that requires ref-counting)
          + still, simple ref-counting is much lower overhead than real heavyweight tracing GC...
      + possibly allow some kind of desugaring of S-expressions into constructs that aren't `CONS` lists?
          + for performance reasons
          + this might involve Hard Compiler Analysis (i.e. how can we reasonably prove that Whatever Purpose This Thing Is Used For would be better served as a vector/array/cons list?)


Possibly ADTs and structs can be introduced with the same syntax (Haskell-style).

Something like the following:

```lisp
(data Point (x: Int y: Int x: Int)) ; struct-style definition
(data Maybe a ((Just a) (Nothing))) ; enum/ADT-style definition
```
It would be fun to allow some kind of special form for ADTs that is a little bit less like s-expression hell; something like:

```lisp
(data Maybe a (Just a | Nothing))
; or, in i-expr form
data 
  Maybe a
    Just a | Nothing
```

### Code generation

Potential targets:
 + LLVM
    - pros:
      * LLVM is ultra-portable
      * well-supported
      * powerful optimizations
    - cons:
      * requires C++ FFI
    - can just `use rustc::lib::llvm;`
      * piggyback on already written rustc LLVM wrappers for C bindings
      * open-source
      * see [iron-kaleidoscope](https://github.com/jauhien/iron-kaleidoscope#llvm-ir-code-generation) for this
    
 + generated C code
    - a la [Nim](http://nim-lang.org/)
    - pros:
      * portability
      * benefit from C compiler optimisations
      * compiler can insert calls to `malloc()`/`free()`
      * easy to link against C libraries
    - cons:
      * in order for the compiler to write good C code, I would have to write _excellent_ C code.
      * making the programmer invoke two separate compilers seems like it might be a productivity issue
         + though the compiler driver could seek out and call the appropriate C compiler
      * this seems like a number of layers of indirection.
    
