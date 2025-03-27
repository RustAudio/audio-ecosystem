We wish to create a new tech stack that will become the new de-facto solution for audio in Rust.

From this new tech stack, we wish to create a new easy-to use high level audio library to cover most generic use case. We could either use the existing `rodio` crate, or we could create a new crate for this.

# The stack
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
