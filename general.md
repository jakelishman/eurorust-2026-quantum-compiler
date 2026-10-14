# Moving a quantum compiler from Python to Rust

## Description (1200 char)

<!--
It will be used as the talk abstract on the conference schedule, if accepted.

General-purpose talks should be accessible to most conference attendees.

We're open to a wide range of topics for general-purpose talks, including but not limited to:
- Rust patterns, anti-patterns and idioms
- Case studies on both industrial and hobbyist projects
- Domain-specific talks (backend, WASM, embedded, system programming, etc.), with minimal prerequisites
- A review of the state-of-the-art in a specific area
- A status report on an ongoing Rust project (or Rust ecosystem) initiative
-->

Over the last decade, quantum computing has moved from a niche research field to an industry already
showing signs of advantage over classical methods in certain areas.  Input quantum programs must be
compiled for specific hardware, which is severely constrained compared to classical hardware,
constantly evolving its instructions sets, and even a fixed processor requires frequent recompilation
due to fluctuating calibrations of the individual instructions.

Qiskit is one of the oldest and most used quantum compilers.  It is fully open source, originating
nine years ago, firmly in the research era of quantum computing.  In this talk, we will walk through
the history of Qiskit's transition from a Python-space project borne out of research work, into a
highly performant Rust-first production-grade compiler and research platform, taking a look at some
of the challenges of quantum compilation and how a steady transition to Rust eased them.


## Talk Outline (unlimited length)

<!--
A detailed overview of the topics your talk will cover, as well as the key
takeaways for the audience.
-->

I'll describe briefly what Qiskit is, some of the tasks of a quantum compiler, and the journey
we took from a pure-Python research project to a Rust-first production compiler.  At each step
of adding more Rust, I'll briefly discuss the algorithm or a data structure in the context of
quantum compilation, so the talk is aiming to show Rust use in an unusual project (for most
people), and include some technical detail on quantum compilation itself.

1.

Briefly: what is a quantum compiler, and what are its tasks?

Quantum computing has very different restrictions to classical computing: no memory except
"registers" (qubits), and registers have very limited interactions with others.  There's no
memory loads/stores, no copying of data, and every operation has a high error rate.

This is generally quite a different proposition to classical compilation, and the focus is
very different.  There is a lot of active research and development, so being open source and
runtime extensible has been very important to the project.

2.

Earliest Rust use: via an encapsulated PyO3 graph library (`rustworkx`)

Our primary compiler IR is a data-flow graph (there's very limited control flow possible in
near-scale hardware, but a lot of parallelism), which used to be Python's `networkx`.  One of our
team began work on a Rust library wrapping `petgraph` in PyO3, leading to immediate 2x speedups in
IR manipulation.

3.

Rust comes to Qiskit: replacing individual passes with Rust

Much of the wall-clock time is spent in a small handful of compiler passes, and some of these
are embarrassingly parallel tasks.  We started replacing the largest bottlenecks by serialising
simplified structures down to Rust, running many trials in threaded parallelism (`rayon`).  The
first of these was solving the "qubit routing" problem, which I'll describe to motivate the move.
This was a transition from compiled Cython to Rust, but even that gave a 5x runtime improvement.

4.

Core data structures move to Rust

We gradually moved the two IRs used in construction and compilation down to Rust entirely, with
tight Python integration.  Moving the data structures to Rust gave us far tighter control over
the memory use, and I'll show a couple of the techniques we used: interning common argument lists;
tagged pointers reducing instruction memory costs; the amount of weight we shed simply by avoiding
`PyObject`.


5.

Separation of Rust from Python

Past a certain critical mass, the mindset shifted from "port it from Python" to "write it in Rust,
provide Python bindings".  We removed all Python objects from core data structures, introduced a raw
C API, and made it possible to write your own compiled extension modules that use our C API.


## Your Expertise (unlimited length)

<!--
Why are you qualified to speak on this topic?

Don't include details that might identify you in the initial anonymous review.
-->

I am the technical lead for one of the few major quantum-computing infrastructure and compiler
projects, which is open source and extensible.  I have a PhD in quantum computing.

I joined [COMPANY] in mid-2021 to work on this compiler, and since then have been the most prolific
contributor to the project, and have been the technical lead of it for the last two years.  I was
one of the driving forces behind the move of our internals from Python to Rust.

## Speaker biography (1200c)

<!-- Same as other submission. -->
