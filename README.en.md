# dsh-voice-mode-plus

An enhanced fork of [dsh-voice-mode](https://github.com/qishuilalala/dsh-voice-mode) for DeepSeek Harness. It keeps the full-duplex voice flow (local streaming ASR, sentence-by-sentence read-aloud with captions, barge-in) and fixes three audible problems, with plugin identities separated so it can coexist with or replace the upstream plugin.

- Sentence pauses: spacing follows the terminal punctuation instead of a flat 20 ms gap.
- Prosody: questions get a slight pitch lift, exclamations a little more volume.
- Local TTS: decimal points survive (`3.14` is read as "three point one four", not a comma).

Independent identity: route `/voice-mode-plus`, settings file `voice-mode-plus.settings.json`, locale namespace `voice-mode-plus`.

## Install

Built artifacts (`lib/`) are committed, so a git install needs no build step:

```sh
dsh plugin --profile <profile> add github:Dgwkuke/dsh-voice-mode-plus
```

Restart the host after installing. Models are reused from `~/.cache/dsh-voice-mode/models` when present.

## Coexistence

Both plugins inject the same microphone slot, so enable only one. To disable the upstream plugin, append to the profile's `cordis.patch.yml`:

```yaml
- id: voice-mode
  disabled: true
```

## Verification

Tested on macOS with dsh `0.2.1-rc.5`: typecheck passes; segmenter 33, verify-client 41, host-security 60, tts-queue-engine 27, emotion 16, tts-playback 8 tests pass; local VITS → SenseVoice round-trip renders "3.14" as "三点十四" (the old logic rendered it without the decimal point). Not verified on Windows/Linux or other dsh versions.

## License

MIT, inherited from upstream. Third-party notices in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Full upstream documentation: [README.upstream.en.md](README.upstream.en.md).
