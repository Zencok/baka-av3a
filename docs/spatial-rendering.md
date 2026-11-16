# Optional spatial processing

Direct `Decoder::decode()` returns multichannel PCM without running spatial effects.
The optional `baka_av3a_core::render` module offers separate building blocks:

| API family | Purpose |
|---|---|
| Bed, object and scene renderers | Map supported channel beds, object positions and scene transport to selected speaker layouts |
| HOA renderers | Convert supported ACN HOA component input to configured speaker layouts |
| Binaural renderers | Process explicitly routed sources or speaker buses through caller-supplied HRIR data |
| Tracking and room processing | Explicit listener-pose and acoustic-path processing for applications that need it |

These APIs do not automatically execute every AV3A static/VR metadata field. They accept
their own validated configuration and reject unsupported routes. Application code selects
which processing, if any, to apply after decoding. See the Rust API documentation for
constructors, supported counts, normalization, delays, tails and reset behavior.

HRIR data is not bundled. Bank preparation and filter/matrix design can be expensive;
perform setup outside the audio callback. Optional processing introduces its own latency
and CPU/memory costs. Validate the selected configuration on your target device rather
than assuming that any supported source/path count meets its audio deadline.

For multichannel device playback, start with the direct decoder output and the device's
required channel mapping. Room geometry, material filters and head tracking are not required.
