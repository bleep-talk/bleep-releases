# Bleep

Connect agents to people on Bleep. CLI and local stdio MCP supply communication;
your agent supplies reasoning.

## macOS developer preview

Apple Silicon, macOS 13 or later:

```sh
brew install bleep-talk/bleep/bleep
bleep --help
```

To update, stop the current host, run `brew update` and
`brew upgrade bleep-talk/bleep/bleep`, then restart. Keep installation state and
configuration outside the Cellar. Do not re-enroll to upgrade. MCP configuration
uses the stable absolute command `$(brew --prefix)/bin/bleep`.

[Quickstart: link, approve and configure Codex](QUICKSTART.md) ·
[Signed/notarized r6 download](https://github.com/bleep-talk/bleep-releases/releases/tag/v0.1.0-r6) ·
[Homebrew tap](https://github.com/bleep-talk/homebrew-bleep)

**Service access remains limited to the approved pilot.** Installation grants
neither backend admission nor Phone permissions. Request access from the Bleep
operator before enrolling a new installation.

## Included

- Encrypted text, photos and AAC/M4A voice notes through CLI and MCP.
- Twelve MCP tools, optional live listen/speak Conversations and idle Content sync.
- Bundled helpers, image/audio validation and the image-safety model.
- A Store lease fix for inherited subprocess descriptors that could cause false
  `Busy` after the original owner ended.

Voice notes support up to 30 minutes and 16 MiB under their accepted signed
policy. Messaging needs no speech-provider credentials. Optional live voice
uses your OpenAI transcription/ElevenLabs speech accounts and operator-supplied
public bridge configuration. No model or teacher is bundled. No Rust, Node,
Python or Codex runtime is required by the communication package.

One host owns an installation at a time. Guided relay setup, automatic steering
into an arbitrary existing desktop chat, and Windows/Linux/Intel packages are
not shipped. Other clients require local stdio MCP and their own configuration;
compatibility is not universal.

## Distribution

All four executables are Developer ID signed with hardened runtime and secure
timestamps, and Apple notarized. Standalone tickets use online Gatekeeper lookup;
offline stapling is not claimed. Checksums accompany the release.

First-party binaries are free to use, including commercially, and may be
redistributed unchanged under the attached Bleep Binary License. Copyright
Maurice Scheffmacher. First-party source remains private. Third-party terms are
unchanged. FFmpeg is a separate executable under
[LGPL 2.1 or later](https://ffmpeg.org/legal.html). Corresponding FFmpeg/Opus
sources, build instructions and notices are included and attached to the release.
