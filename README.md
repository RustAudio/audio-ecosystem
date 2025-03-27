Currently, the audio ecosystem in Rust is quite fragmented. There are many different audio-related crates each with different goals and approaches. See our overview of the [current audio landscape](current_audio_landscape.md). We also feel that this ecosystem is currently limited in its capabilities.

In short, the goal is to unify many of these efforts into a single organization with a shared vision, so that we can create *the* de-facto solution for audio in Rust that is both powerful and easy to use.

# Why do this?
- Sharing knowledge between developers will help to create the best possible experience and set of features for users.
- Having a shared organization will ease the burden on maintainers, and lessen the chance of a project being abandoned.
- Having a single solution with multiple active maintainers will increase user's confidence in these libraries.

# How will we do this?
- Chart the features needed by developers using audio, see:
  [requirements](requirements.md)
- Define a de-facto tech stack for audio in Rust, see [techstack](techstack.md).
- Move the key parts of the stack into a shared organization 
- Build a single high level crate that handles most use-cases without requiring
  audio knowledge

# Current state
We are at an early beginning. The text on this repo is a work in progress, it should be seen as a draft. It will change over time with more input.
