# Writing interdependent Python extension modules in Rust with PyO3

## Description (1200 chars)

<!--
It will be used as the talk abstract on the conference schedule, if accepted.

For the deep dives, assume that the audience has some familiarity with the
domain or ecosystem you're working with. For example, if you are submitting an
embedded deep-dive, you might assume that the listener has written at least
some embedded code (in Rust or another language) and is aware of the key crates
in the Rust ecosystem for embedded development (e.g., embassy).

We're open to a wide selection of topics along the same vein as the
general-purpose talk topic listed above, but would be particularly pleased to
see submissions in the following areas:

- Using Rust for data processing, ML training and/or inference
- Using Rust in an industrial embedded project
- Using Rust for backend or GUI development
- Embedding Rust code in a larger codebase written in another language (or vice-versa)
- A deep dive into a certain language feature or the internals of a popular crate
- Using Rust to write kernel drivers (or of a full-blown operating system!)
-->

Rust has built-in tooling for defining C-callable APIs, and a strong ecosystem
around exposing libraries to C (via `cbindgen`) and Python (via PyO3).  One
major pain point in Python-extension development with Rust is the difficulty of
allowing downstream extension modules to interact with the Rust code of a
foundational package; Rust's compilation story for shared libraries is already
challenging, with Python's dynamic runtime environments and linking
requirements only increasing the complexity.

This talk will show how we used PyO3 and `cbindgen` to rewrite [PROJECT], the most widely
used quantum-computing compiler and framework, from being pure Python into
a Rust core with a C API and bindings to Python.  I'll show how we expose the C
API from our Python-space package for other Rust-based Python packages to use
via PyO3, without compiling or linking against an unstable Rust ABI.  I'll talk
about how this enabled other [COMPANY] teams to solve performance bottlenecks within our
stack, while maintaining clean separations of concerns.


## Talk Outline (no length)

<!--
A detailed overview of the topics your talk will cover, as well as the key
takeaways for the audience.
-->

I'll be assuming some familiarity with C and Python, and some familiarity with
PyO3, but not any detailed knowledge of how (C)Python makes extension modules
work nor the Python C API (neither in raw C nor low-level `pyo3-ffi`).  There is
no knowledge necessary on quantum computing or quantum compilation; our package
is just the case study in our techniques, not a starring player.

Note that a "Python extension module" specifically refers to compiled-language
extensions using Python's C API, and not a pure-Python package.

Topics:

- A quick hand-wavy overview of the job of our quantum-compilation framework, to
  motivate its history, its need for dynamic extensions, and why we took our
  particular incremental path to porting to Rust and providing a C API.

  I'll talk about our journey from pure Python to predominantly Rust with
  Python and C wrapping: first, porting individual bottlenecked algorithms;
  second, moving core data structures from Python to Rust; third, exposing C-API
  access in Python-free builds.

  I'll pose the question of how to expose this same C API to _other_ Python
  extensions, themselves written in Rust.


- Briefly, what is provided by PyO3, and the interaction limitations
  inherent to Rust's compilation model when mixed with Python extension modules.

  PyO3 provides excellent ergonomic Rust bindings to Python's C API, but can't
  overcome the limitation that two separately compiled crates, even sharing the
  same Rust-space dependencies, in general do not have a shared ABI; it is not
  practical to pass ergonomic Rust objects from one PyO3 extension module to
  another.

  Takeaway: there are limitations in Rust's compilation story that prevent
  writing multiple Python extension modules that can interact with each other
  via Python space.


- Briefly, how Python extension modules work, and how they can (kind of) link
  against each other for compiled-language extensibility.

  Python extension modules are not permitted to directly link against other
  extension modules at compile time (at least, not if you want them to be safely
  distributable or separately upgradable).  Instead, the Python C API provides a
  way of safely passing raw pointers between modules, and it is up to individual
  modules to use this to share APIs.

  Takeaway: it's possible to write two Python extension modules that interact
  with each other via C APIs, but the linking story is deeply unpleasant for
  natural Rust use.


- A description of our architecture and implementation for allowing downstream
  PyO3 extension modules accessing our C API.

  First: we store `PyCapsule`s containing pointers to "vtables" (function
  pointers at known, static offsets) of the C API functions in our first-party
  foundational module; these are generated in `build.rs` with a combination of
  `cbindgen` and custom tooling.  Second: we define a (generated, again
  `cbindgen`-derived) `<project>-pyo3` "raw FFI" crate akin to a `<project>-sys`
  crate for system-library linking, which defines all the C-API functions as
  Rust-native ones calling into the correct runtime-linked vtable offsets.
  Third (and optionally): ergonomic Rust-space wrappers around the "raw FFI"
  crate, which downstream crates can use.

  I'll briefly discuss the amount of custom tooling we needed to fit all this
  together, including to maintain ABI compatibility in the pointer offset tables
  between different Python-package versions (i.e.: how we stop ourselves
  accidentally shifting the offsets around by accident when adding new
  features).  I'll give examples of how this is used within [COMPANY].

  Takeaway: we've got a production-tested pattern for writing inter-dependent
  Python extension modules in Rust.  There's still more custom tooling than we'd
  like, and potential to work with the community to make more of our
  PyO3/`cbindgen`/custom infrastructure reusable.

(I'm aware that Luca Palmieri is giving a tutorial on C APIs, and gave a talk
at RustConf Seattle 2025.  This talk proposal is very much complementary to
Luca's previous talk; the main focus is on additional complexities when dealing
with Python, and Rust talking to Rust via C.)

## Your Expertise (no length)

<!--
Why are you qualified to speak on this topic?

Don't include details that might identifty you in the initial anonymous review.
-->

I am the technical lead for one the few major quantum-computing infrastructure
and compiler projects, which is open-source and extensible.  I led my team to
rewrite its internals from Python to Rust, and then expose these for
cross-language extensibility via its C API, leading to order-of-magnitude
speed-ups in compilation, and allowing extensibility from other compiled
languages.

I led the team to design and implement the infrastructure around making this
compiled-language extensibility to other Python-package authors, which is the
primary way users interact with our software; our Rust internals are private
implementation (but still open source!), and "compile against our Rust crates"
is not generally a useful option.  I worked with other [COMPANY] teams writing
software on top of ours to define their own C APIs from Rust code in similar
manners, and to write tooling on top of ours via Python extensions to solve
critical performance bottlenecks in their parts of the stack.

## Speaker biography (1200 chars)

Jake Lishman is a senior research software developer at IBM Quantum. He is
currently the technical lead of the open-source Qiskit SDK, the most widely used
compiler and framework for quantum computing.

Jake started in open-source software in early 2020 with QuTiP, a Python package
for quantum-physics simulations with Cython linear algebra internals, while
finishing postgraduate studies in quantum computing.  He joined IBM Quantum in
2021, working on Qiskit and the programming-language design and tooling for
OpenQASM 3, a domain-specific language for expressing quantum computation.
