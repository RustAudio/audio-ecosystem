Currently, the audio ecosystem in Rust is quite fragmented. There are many different audio-related crates each with different goals and approaches. We also feel the this ecosystem is currently limited in its capabilities.

In short, the goal is to unify many of these efforts into a single organization with a shared vision, so that we can create *the* de-facto solution for audio in Rust that is both powerful and easy to use.

# Why do this?
- Sharing knowledge between developers will help to create the best possible experience and set of features for users.
- Having a shared organization will ease the burden on maintainers, and lessen the chance of a project being abandoned.
- Having a single solution with multiple active maintainers will increase user's confidence in these libraries.

# Requirements

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
Bevy is a widely used game engine build around the ECS model. Because bevy uses ECS it has specific demands from an audio engine.

*TODO*

### Supported by Kira
Kira is widely used, a unified audio stack needs to at least have feature
pairty with Kira.

*TODO*

### Added by unification team
- Change speed without changing pitch @dvdsk
- Switch audio device without gap. This would enable writing VOIP applications. There it is usual for users to want the VOIP audio to go through another output (headphones) then the system default. @dvdsk
- Microphone as input node to the audio graph @dvdsk. 
- Add an output to the OS, any audio send to it enters the audio graph. @dvdsk
- Add an input to the OS, it gets audio from an output of the audio graph. @dvdsk
