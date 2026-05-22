# BTM — Beat-Tracking Media

**BTM** is a family of backward-compatible file format extensions that enable *musical structure* of an animation or video — how many beats it spans, and which frames coincide with beats — instead of baking a specific playback tempo into frame timing. A BTM-aware player can then retempo the content in real time to match the beat of accompanying music. The metadata sits inside the host container as a standard extension block, so BTMA files remain valid GIFs and BTMV files remain valid MP4s — no extension change required, no breakage in non-aware viewers.

## The problem

Animated emotes on livestream platforms, looping background visuals on DJ streams, music-video clips, and other rhythmic media are typically stored in formats (GIF, MP4) that encode their playback speed as fixed per-frame timing. When the music playing alongside that media is at any tempo other than the one the file was encoded for, the visual drifts out of sync — subtly at first, badly over time.

The visual content of these files is usually correct; only the encoded playback speed is wrong for any given live context. BTM fixes this without changing the visual content.

## The approach

BTM defines a small metadata block, `BTM1Sync`, that records:

- A **default BPM** for playback when no external tempo is detectable by, or sent to, a BTM renderer
- An ordered list of **beatmarkers** (frame indices that should land on beats)

This metadata block is embedded inside an existing container format using that container's standard extension mechanism. The result is a file that:

- Plays normally in any non-aware viewer at its encoded fallback tempo
- Retempos to live music in any program/extension/plug-in that has a BPM source (live beat detection on the audio bus, MIDI clock from a DJ controller, Ableton Link, or a manual input)

Because the file is structurally identical to the host container with one extra metadata block, existing tooling treats BTM files as ordinary files of that format. No new codec is needed, no new MIME type, no platform cooperation required to ship a working file. Platforms that do not accept uploads of a differing filename extension are usable by changing the extension to match, but it is unknown whether any given platform will also strip BTM-specific metadata from the uploaded file. This README will be updated with a list of platforms that are confirmed to do this. 

## The format family

BTM is a family with shared metadata semantics and per-container profiles:

| Profile | Container | Use case | Extension |
|---------|-----------|----------|-----------|
| **BTMA** | GIF89a | Animated chat emotes, small looping animations | `.btma` |
| **BTMV** | MP4 (ISO BMFF) | Stream visuals, music-video loops, VJ content | `.btmv` |

Both profiles share the same `BTM1Sync` metadata block and the same playback rules. They differ only in how the block is embedded in the host container (a GIF Application Extension block for BTMA, an MP4 UUID box for BTMV) and in container-specific concerns (color depth, codec choice, seeking behavior).

Future profiles for other containers (WebM, APNG) may be defined in later revisions.

## Specifications

The v0.1 specification consists of three documents in the [`spec`](https://github.com/btm-format/spec) repository:

- **[`BTM1Sync-schema-v0.1.md`](https://github.com/btm-format/spec/blob/main/BTM1Sync-schema-v0.1.md)** — The shared metadata schema. Container-independent. Defines the binary layout of the `BTM1Sync` block and the playback semantics that all conforming implementations must follow.
- **[`BTMA-spec-v0.1.md`](https://github.com/btm-format/spec/blob/main/BTMA-spec-v0.1.md)** — The GIF profile. Specifies how the `BTM1Sync` block is embedded in a GIF89a Application Extension, the GIF-fallback frame delay computation, and the GIF-to-BTMA conversion workflow.
- **[`BTMV-spec-v0.1.md`](https://github.com/btm-format/spec/blob/main/BTMV-spec-v0.1.md)** — The MP4 profile. Specifies the BTM1Sync UUID box layout, presentation-order frame indexing, codec recommendations, and the MP4-to-BTMV conversion workflow.

The specifications are released under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/). Anyone is free to implement BTM, build tools around it, redistribute the spec text, or fork the format for their own purposes; attribution to this repository is requested.

## Status

**v0.1 — Draft.** The specifications are stable enough to implement against, but no reference implementations exist yet, and no production deployments have been validated. The v0.1 → v1.0 transition will occur once at least one working implementation has been built and tested end-to-end. Breaking changes between v0.1 and v1.0 are possible but unlikely; the schema's wire format and playback semantics are intended to remain compatible.

## A motivating example: the DJ stream emote problem

A DJ streaming on Twitch typically has a custom emote library — animated GIFs of disco parrots, moshing ducks, and any kind of cat. These emotes were authored at specific tempos (that can vary to a large degree) and encoded with fixed frame delays to play at those tempos.

When the DJ plays a track, mostl likely more than a few emotes on screen are subtly out of sync. Multi-beat-loop emotes (2-beat, 4-beat) drift against the music's phrasing across each loop.

A DJ converting these emotes to BTMA needs to provide one piece of information per emote: how many beats does this loop span? The conversion tool reads the GIF's existing duration, computes a sensible default BPM, places that many beatmarkers evenly, and inserts the metadata block. The visual content is untouched. The file remains a valid GIF that plays at the original speed in any GIF viewer.

In a BTMA-aware viewer with a BPM source (live beat detection on the stream audio, for instance), the emote retempos itself in real time. Whatever tempo the DJ is mixing at, every BTMA emote on screen is locked to it.

## Reference implementations

Coming. The intended initial deliverables are:

- A GIF-to-BTMA converter (the "how many beats?" workflow)
- A browser-based BTMA renderer that performs live beat detection via the Web Audio API
- An MP4-to-BTMV converter
- An OBS plugin that renders BTMV files synced to the broadcast audio bus

These will live in separate repositories under this organization as they're built.

## Contributing

The format is at a stage where structural feedback is more valuable than line edits. If you're implementing BTM and encounter ambiguity in the spec, please open an issue on the [`spec`](https://github.com/btm-format/spec/issues) repository describing the case and what behavior would be most useful. The goal for v1.0 is a spec that an implementer can build from in an afternoon without needing to ask any questions.

If you're a DJ, VJ, livestreamer, emote artist, or platform engineer whose work this format would affect, real-world feedback on the use case (does the conversion workflow match how you'd actually want to convert your library? are the beat counts and BPMs you encounter compatible with the schema? what's missing?) is especially welcome.

## Background

BTM was conceived as a way to fix the specific problem of fixed-tempo animated emotes on DJ streams, and grew into a more general format family during design discussions. The full design rationale, including alternatives considered and tradeoffs made, is preserved in the spec documents' non-normative sections.
