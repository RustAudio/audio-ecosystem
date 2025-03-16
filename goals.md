We believe all audio needs can be shared by one base. Currently there are
multiple audio engines. They were all made with different goals in mind.

# Why
- eases the burden on maintainers.
- speeds up development.
- lessens the chance of a project being abandoned.

# How
We hope to achieve this:
- demonstrating an existing crate as a good base.
- building a crate on top of that existing crate to serve other use cases.
- combining the best ideas and code from existing projects.

# What
- a crate being a good base: We can implement all the needed use cases with
  good performance.
- good performance: maintaining low latency without audible audio glitches. This
  requires processing time to be consistent. 
