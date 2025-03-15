We believe all audio needs can be shared by one base. Currently there are
multiple audio engines. They all where made with different goals in mind.

# Goals
We aim to:
- ease the burden on maintainers.
- speed up development.
- lessen the chance of a project being abandoned.

# How
We will do this by:
- demonstrating an existing crate as a good base.
- building a crate on top of that existing crate to serve other use cases.
- combining the best ideas and code from existing projects.

# Definitions
- performing well: maintaining low latency without audible audio glitches. This
  requires processing time to be consistent. 
