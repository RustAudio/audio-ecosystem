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
Bevy is a widely used game engine build around the ECS model. Because bevy uses ECS it has specific demands from an audio engine.

*TODO*


### Added by unification team
Needs we have ourselves or features we have seen requested but can not link too.

- Change speed without changing pitch @dvdsk
- Switch audio device without gap. This would enable writing VOIP applications. There it is usual for users to want the VOIP audio to go through another output (headphones) then the system default. @dvdsk
- Microphone as input node to the audio graph @dvdsk. 
- Add an output to the OS, any audio send to it enters the audio graph. @dvdsk
- Add an input to the OS, it gets audio from an output of the audio graph. @dvdsk
- Have (partial) functionality on `no_std` targets. Such as capable embedded hardware. The targets need to at least:
    - Can supports 32 bit floating points through [libm](https://crates.io/crates/libm).
    - Has 32 bit float atomics via [portable-atomics](https://crates.io/crates/portable-atomic).
  This would allow applications such as: 
    - (battery operated) Alarm clocks
    - Light portable audio players
  We might make this a soft requirement, meaning we attempt to achieve this if it does not take too much extra effort.
