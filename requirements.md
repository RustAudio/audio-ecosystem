# Requirements
Becoming the standard audio stack for Rust requires us to support near all needed use-cases. This is an attempt to list as many as we can. 

To that end we start with features supported by popular Rust audio engines like *Kira* & *Rodio*. Requests of those engines also point to needed use-cases. Finally we hope to learn from looking at large projects using audio like *Bevy* and *Fyrox*.

### Supported by Kira
- Smoothly adjusting properties of sounds without pops (via [tween](https://docs.rs/kira/latest/kira/struct.Tween.html))
- Change the settings of effects automatically [modulate](https://docs.rs/kira/latest/kira/modulator/index.html)
- Apply audio effects: 
  - Compression
  - Delay
  - Distortion
  - Equalizer
  - Filter
  - Stereo panning
  - Reverb
  - Volume control
  - Fade-in/Fade-out (via tweens)
- Add custom effects
- Control audio effects from anywhere in the program
- Control the application of an effects on multiple tracks from one point
- Flexible mixer for applying effects to audio, 
- A clock system which allows starting sounds at precise times, ideal for rhythm games. 
- Loop a specific range of a piece of audio
- Seek to a specific position in the audio 
- Control the pitch of audio in musical intervals (semitones) though this also
  affects playback rate
- Spatial audio (volume/panning based), can link effect parameters to distance
  from listener

### Supported by Rodio

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

Bevy is a popular game engine built around the ECS paradigm.
Its ECS is highly flexible, so there are many valid ways to integrate
audio engines. However, idiomatic integrations
should strive to satisfy a few conditions:

1. ECS-driven design.
2. Broad platform support.
3. Integration with Bevy's asset system.

Bevy is looking for a _first-class_ audio solution.
Playing samples, applying effects, routing audio, and
even authoring effects should feel natural, not tacked-on.
Adhering to these conditions will help guide engines and integrations
towards that first-class experience.

#### ECS-driven design

Increasingly, crate authors within Bevy's ecosystem are choosing to integrate
deeply with the ECS; they are choosing _ECS-driven_ design.
This provides quite a few advantages to both crate authors and users.
For example, keeping track of emitter positions for spatial audio is easy
when emitter and transform components share the same entity.
And, with each release, the capabilities of the ECS are only expanding.

Luckily, audio engines tend to mesh well with the ECS. Playing sounds
can be modeled by spawning sound entities. Audio tracks and buses
can be modeled as groups of related entities. Depending on the
engine, it may also make sense to model effects as entities.

To facilitate this design, user-facing types _must_ implement
Bevy's [`Component`](https://docs.rs/bevy/latest/bevy/ecs/component/trait.Component.html)
trait. Note that `Component` requires `Send + Sync + 'static`.
Ideally, core types should implement `Component` directly, not via wrapper or
proxy types. Audio engines can feature-gate their `Component` implementations
to maintain engine-agnostic design.
Most audio engines will likely need targeted abstractions over
_some_ parts of their API surface, but these should be kept minimal.

#### Broad platform support

Bevy features broad platform support, including
desktop platforms, mobile, WebAssembly, and increasingly
resource-constrained `no_std` environments.
Users will expect audio engines to provide similarly broad
support either out of the box or with extensible backends.
Core functionality like sample playback should not depend
on the ability to read files from disk.

#### Integration with Bevy's asset system

Audio engines must be able to integrate with Bevy's
asset system. The asset system helps maintain cross-platform
compatibility, de-duplicates already-loaded assets, and
automatically handles loading and decoding in separate threads
on supporting platforms.

To support this integration, the only requirement is that
audio engines must accept in-memory sample sources.

### Added by unification team
Needs we have ourselves or features we have seen requested but can not link too.

- Change speed without changing pitch @dvdsk
- Switch audio device without gap. This would enable writing VOIP applications. There it is usual for users to want the VOIP audio to go through another output (headphones) then the system default. @dvdsk
- Microphone as input node to the audio graph @dvdsk. 
- Add an output to the OS, any audio send to it enters the audio graph. @dvdsk
- Add an input to the OS, it gets audio from an output of the audio graph. @dvdsk
