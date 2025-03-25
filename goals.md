Currently, the audio ecosystem in Rust is quite fragmented. There are many different audio-related crates each with different goals and approaches. We also feel the this ecosystem is currently limited in its capabilities.

In short, the goal is to unify many of these efforts into a single organization with a shared vision, so that we can create *the* de-facto solution for audio in Rust that is both powerful and easy to use.

# Why do this?
- Sharing knowledge between developers will help to create the best possible experience and set of features for users.
- Having a shared organization will ease the burden on maintainers, and lessen the chance of a project being abandoned.
- Having a single solution with multiple active maintainers will increase user's confidence in these libraries.

# An overview of the current Rust audio ecosystem

First we will take a look at the current landscape of highly-used Rust audio crates, as well as some places where we believe improvement is needed.

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

# The new tech stack

We wish to create a new tech stack that will become the new de-facto solution for audio in Rust.

This proposed tech stack would comprise of the following crates:

## [Firewheel](https://github.com/BillyDM/Firewheel)

At the heart of project is the "audio graph" engine. An audio graph allows users to connect any combination of audio streams, sound generators, and audio effects together in any configuration. It provides the low-level tools needed to create almost any audio solution (*aside from say a full-on digital audio workstation). Think of it like [wgpu](https://wgpu.rs/), but for audio.

We plan to use [Firewheel](https://github.com/BillyDM/Firewheel) as our audio graph engine. It has been designed from the ground-up using the lessons learned previous engines.

Some of its key features include:
- Modular design that can be run on any backend that provides an audio stream
- Flexible audio graph engine (supports any directed, acyclic graph with support for both one-to-many and many-to-one connections)
- A suite of essential built-in audio nodes
- A custom audio node API that allows for an ecosystem of 3rd party generators and effects
 - Basic [CLAP](https://cleveraudio.org/) plugin hosting (TODO) (non-WASM only), allowing for more open source and proprietary 3rd party effects and synths
- A built-in event system for syncing state, along with macros to reduce state synchronization boilerplate.
- Silence optimizations (avoid processing if the audio buffer contains all zeros, useful when using "pools" of nodes where the majority of the time nodes are unused)
- Support for loading a wide variety of audio files using [Symphonium](https://github.com/meadowlarkdaw/symphonium).
- Properly respect the hard realtime constraints of audio (no mutexes!)
- An API that is friendly to ECS's (entity compoment systems), which makes it play nicely with the [Bevy]() game engine.
- WebAssembly support

### Non-Goals of Firewheel

While Firewheel is meant to cover nearly every use case for games and generic applications, it is not meant to be a complete DAW (digital audio workstation) engine. Not only would this greatly increase complexity, but the needs of game audio engine and DAW audio engine are in conflict*.

> \* This conflict arises in how state is expected to be synchronized between the user's state and the state of the processor. In a DAW, the state is tied to the state of the "transport", and the host is allowed to discard any user-generated parameter update events that conflict with this transport state (or vice versa). However, in a game engine and other generic applications, the user's state can dynamically change at any time. So to avoid the processor state from becoming desynchronized with the user's state, parameter update events are only allowed to come from a single source (the user), and are gauranteed to not be discarded by the engine.

* MIDI on the audio-graph level (It will still be possible to create a custom sampler/synthesizer nodes that read MIDI files as input.)
* Parameter events on the audio-graph level (as in you can't pass parameter update events from one node to another)
* Connecting to system MIDI devices (Although this feature could be added in the future if there is enough demand for it).
* Built-in synthesizer instruments (This can still be done with third-party nodes/CLAP plugins.)
* Advanced mixing effects like parametric EQs, compressors, and limiters (This again can be done with third-party nodes/CLAP plugins.)
* GUIs for hosted CLAP plugins (Although this feature could be added in the future if there is enough demand for it).
* Multi-threaded audio graph processing (This would make the engine a lot more complicated, and it is probably overkill for games and generic applications.)
* VST, VST3, LV2, and AU plugin hosting

### [Interflow](https://github.com/SolarLiner/interflow)

While [CPAL](https://github.com/RustAudio/cpal) has served the Rust ecosystem well over the years as a platform-agnostic audio input/output library, its API does not allow for essential features like duplex audio devices. Some have found its API to be cumbersome and confusing.

As such, we want to see if this new library provides a better alternative. It aims to address the problems we have had with CPAL, as well as being able to support native duplex audio devices.

### [fixed-resample](https://github.com/MeadowlarkDAW/fixed-resample)

An easy to use crate for resampling at a fixed ratio. It uses [Rubato](https://github.com/HEnquist/rubato) internally.

This crate also contains a realtime-safe spsc channel type that automatically resamples the input stream to match the output stream when needed. This is needed to sync audio streams together, or to stream audio to/from a network stream.

### [Symphonium](https://github.com/MeadowlarkDAW/symphonium)

An easy-to-use wrapper around [Symphonia](https://github.com/pdeljanov/Symphonia) for loading audio files into memory. (It also uses `fixed-resample` for resampling.)

Outlined here are some other key repositories that I believe should also be added to this new organization:

### [creek](https://github.com/MeadowlarkDAW/creek)

Realtime-safe streaming to/from audio files on disk. It uses `Symphonia` internally. (Note that `fixed-resample` and `fast-interleaved` are in the works to be dependencies of creek).

### [RtAudio-rs](https://github.com/MeadowlarkDAW/rtaudio-rs)

Safe Rust wrapper and bindings to [RtAudio](https://github.com/thestk/rtaudio). This can be used as an alternative backend to CPAL and Interflow if needed.

### [fast-interleave](https://github.com/MeadowlarkDAW/fast-interleave)

Fast and easy-to use interleaving and de-interleaving methods in Rust. Used by `fixed-resample`, `Symphonium`, and `Firewheel`.

### [audio-channel-buffer](https://github.com/BillyDM/audio-channel-buffer)

A WIP collection of standard audio buffer types that are performant and memory-efficient. This one needs more work before it should be used, but it would be helpful to have a single standard for audio buffer types to use across our crates.

## Essential libraries to ask for inclusion

Since the following crates are an essential part of the stack, we should ask their respective developers if they would be interested in including their crates in this new organization as well. (Though it's not a big deal if they say no.)

### [Symphonia](https://github.com/pdeljanov/Symphonia)

A pure Rust audio decoding and media demuxing library supporting AAC, ADPCM, AIFF, ALAC, CAF, FLAC, MKV, MP1, MP2, MP3, MP4, OGG, Vorbis, WAV, and WebM.

If they maintainers do say yes to including it in the new organization, then it might be better to make the `Symphonium` wrapper an official part of `Symphonia`.

### [Rubato](https://github.com/HEnquist/rubato)

An audio sample rate conversion library for Rust.

# A new de-facto high level audio library

From this new tech stack, we wish to create a new easy-to use high level audio library to cover most generic use case. We could either use the existing `rodio` crate, or we could create a new crate for this.

### Currently supported features of Rodio

This new solution will need to support the already existing feature of `rodio`.

Rodio has quite a large user base Therefore its safe to assume that its current features are therefore needed.

* Various effects: Amplify, High/Low pass, Fade in/out, Speed changes, Audio normalization
* Ability to queue sounds and mix them together
* Generate sound (chirp/sine/square/triangle/sawtooth)
* Skip, stop/abort, and pause audio samples
* Seek to a given duration from the start of an audio sample
* Convert between channel counts adhering to dwChannelMask. (does anyone know if that is a standard and if such which?)
* Mix mono to spatial audio

### Requested by Rodio users

These use-cases have been requested by Rodio users here: [User stories/usecases/experiences](https://github.com/RustAudio/rodio/issues/626).

- Sample perfect scheduling, *express start playing this sound at sample index i=xxx*. [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2425839614)
- Sample perfect looping, *loop sounds without any gap in the audio output*. [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2425839614)
- Low-latency scheduling, *key-presses triggered certain sounds*. [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2425839614)
- Crossfade, *audio in a queue where items can still be swapped*
- Managing channels. [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2470118600)
  - *take audio only from certain channels*
  - *customize up/down mixing strategy (mono -> 5.1 for example)*
  - *output channel information available in the audio graph*
- Dynamic limiting [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2593811346)
- Support large playlists/queues, *can not open all sources (files) simultaneously* [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2599643841)
- Query the total duration of playing sound, *with distinction between unknown and infinite* [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2599643841) [context](https://github.com/RustAudio/rodio/issues/626#issuecomment-2661094585)
- Callback when a piece finishes playing [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2599643841)
- Follow default output device without gap [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2599643841)
- Easy to contribute effects/sources to [user story](https://github.com/RustAudio/rodio/issues/626#issuecomment-2599643841)

### Planned/In development by Rodio team:
These are mostly in response to trends we see in issues made

- Single struct that supports all music players use cases and is trivial to use. *No design work has been done yet, would replace rodio's [Sink](https://docs.rs/rodio/latest/rodio/struct.Sink.html) struct*
- ChannelRouter, every output channel is the weighted sum of all the input. Weights can be changed during playback. [design issue](https://github.com/RustAudio/rodio/issues/653)
- Give all the effects optional controls that can be freely moved through programs. (Clone + Send + Sync) [discussion](https://github.com/RustAudio/rodio/issues/658)
- Switch between slower HiFi resampler and a fast LoFi one.

### Needed by Bevy

*TODO*

### Supported by Kira

*TODO*

### Added by unification team
- Change speed without changing pitch @dvdsk
- Switch audio device without gap. This would enable writing VOIP applications. There it is usual for users to want the VOIP audio to go through another output (headphones) then the system default. @dvdsk
- Microphone as input node to the audio graph @dvdsk. 
- Add an output to the OS, any audio send to it enters the audio graph. @dvdsk
- Add an input to the OS, it gets audio from an output of the audio graph. @dvdsk
