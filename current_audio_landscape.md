# Overview of the current audio landscape

Let's take a look at the current landscape of highly-used Rust audio crates, as well as discuss places where we believe improvement is needed.

### [Rodio](https://github.com/rustaudio/rodio)
Rodio is currently the most popular one-stop solution for audio playback. This is most likely due to its ease of use.

While rodio has served the Rust ecosystem well, its API is inherently limiting, preventing us from adding features for more advance use cases beyond just simple audio playback with simple effects.

### [CPAL](https://github.com/RustAudio/cpal)
CPAL provides access to system audio input and output device streams with a platform-agnostic API. This is currently used by almost all Rust projects that provide audio input and output, including `Rodio`, `Kira`, and `Fyrox`.

While it currently works great when connecting to a single output device, it currently does not handle "duplex audio" well. "Duplex" means connecting to both an audio input device and audio output device at the same time, a common use case. Many OS audio APIs and audio devices natively support duplex audio streams which are optimized to have as little latency as possible. CPAL's API currently requires the user to manually sync audio streams together, which is difficult, error-prone, and inherently introduces a bunch of latency. Adding native duplex support to CPAL would require an overhaul of the API.

In addition, many users including myself have found its API for enumerating device capabilities overly cumbersome.

### [Kira](https://github.com/tesselode/kira)

A highly capable and expressive audio engine for games.

We have found that its API is bit too high-level and opinionated for use as a general-purpose audio engine, and that its API does not mesh very well with ECS-based game engines like Bevy.

### [Fyrox](https://github.com/FyroxEngine/Fyrox)
Fyrox is a fully-featured Rust game engine, and not a library for audio. However, it does include its own custom audio engine, so I felt it was necessary to include here.

Fyrox's audio engine is custom-tailored for games, and includes the ability for advanced effects.

### [Oddio](https://github.com/Ralith/oddio)

A small and lightweight audio engine for games.

Like Kira it is a bit too high-level and opinionated for use a general-purpose audio engine. It also hasn't been updated in over 2 years.

### [Symphonia](https://github.com/pdeljanov/Symphonia)
Symphonia is a decoder for many popular audio file formats.

It works great, however its API is too low level for most users. We feel that there needs to be an easy-to-use abstraction built on top of it.

### [Rubato](https://github.com/HEnquist/rubato)

An audio sample rate conversion library for Rust.

It works great, however we feel there needs to be an easy-to-use API to handle the common use case of resampling at a fixed ratio (i.e. when loading audio files from disk that have a different sample rate from the project's sample rate).

### [Hound](https://github.com/ruuda/hound)

A simple library that decodes and encodes audio files in the WAV audio file format.

There is nothing about this library that we feel needs improvement. I just felt the need to add it here since it is so widely used.

