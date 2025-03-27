Currently Rodio is by far the most widely used audio engine. Unfortunately it
seems limited by what we knew 9 years ago about audio development in Rust. Since
then new crates have been build to overcome that. We hope to demonstrate one
crate to be best suited to all use cases and then build support behind it. For
that we are building a demo. It will take the form of a game environment since
game audio is seen as the most challenging use case. The goal is to have users 
feel how the game responds differently when switching audio engine.

Currently two crates are participating in this demo: `Rodio` and `Firewheel`
though everyone is very welcome. The demo will be limited to functionality that
is properly implemented by all crates.

# Demo 
- audio environment playing back many sounds at the same time.
- sounds and effects are influenced by user input to make it easy to judge
  latency.
- the demo uses audio libraries in the same way as a game.
- pre-defined events instead of user input to keep the demo repeatable.
- no graphics to keep the demo code simple.
- sounds are all at the same sample-rate.
- implementations pre-resample before loading.
