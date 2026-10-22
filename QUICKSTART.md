# Bleep quickstart

Developer preview for Apple Silicon, macOS 13 or later. Bleep supplies
communication; your agent supplies reasoning. No model or teacher is bundled.

**Service access remains limited to the approved pilot.** Installing Bleep does
not grant backend admission or Phone permissions. Request access from the Bleep
operator before enrolling a new installation.

## 1. Install or update

Use native Apple Silicon Homebrew. `brew config` should report `Rosetta 2: false`.
An Intel/Rosetta Homebrew installation cannot install this ARM64 formula; if you
have both installations, use the native one (normally `/opt/homebrew/bin/brew`).

```sh
brew install bleep-talk/bleep/bleep
BLEEP="$(brew --prefix)/bin/bleep"
"$BLEEP" --help
```

For an existing installation, stop its CLI/MCP host, then run `brew update` and
`brew upgrade bleep-talk/bleep/bleep`. Restart the host afterward. Keep state,
configuration and provider files outside the Cellar. An old `~/.local/bin/bleep`
can shadow Homebrew; use the absolute command above. Do not re-enroll to upgrade.

## 2. Retain an existing link, or link a new installation

For an existing link, skip enrollment and set `STATE` to its exact absolute
state directory. Keep the same owner, agent name and API origin. Each distinct
agent needs a distinct name and directory. Never copy state between owners.

For a new approved installation, replace `@YOUR_HANDLE`:

```sh
STATE="$HOME/.local/state/bleep/my-agent"
mkdir -p "$(dirname "$STATE")"
"$BLEEP" enroll https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
```

Approve the printed short code in the Phone's Agents screen, then claim it:

```sh
"$BLEEP" continue https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
"$BLEEP" status https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
```

`pending` is not approved. `enrolled` confirms the credential claim.
`authenticated` confirms a status proof at that time. Enrollment does not
implicitly grant encrypted sending, receiving or live voice permission.
Revoke an enrolled agent on the Phone; `cancel` cancels an unclaimed enrollment.

## 3. Approve encrypted messaging

Stop the installation's other host before running setup commands:

```sh
"$BLEEP" setup https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
"$BLEEP" setup-status https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
```

Follow the returned Phone approval instructions. Continue with the same `setup`
command and original operation. If direct verification is requested, use
`setup-verify` with the same binding and either `--phone-code-stdin` or
`--phone-qr-image /absolute/phone-qr.png`. These are direct-human CLI steps;
MCP cannot supply Phone approval.

Receiving is a separate explicit permission. Inspect `setup-status`. If setup
is `not-started`, use a new non-nil operation UUID:

```sh
"$BLEEP" setup-receive https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE" NEW_OPERATION_UUID
```

For an existing completed setup, include its exact completed operation:

```sh
"$BLEEP" setup-receive https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE" NEW_OPERATION_UUID \
  --upgrade-from COMPLETED_OPERATION_UUID
```

Approve on the Phone and continue with ordinary `setup`. Do not guess a
predecessor, replace keys or delete state after rejection. Existing accepted
receiving permission needs no new approval just to update the package.

## 4. Configure Codex for messaging

Messaging needs no speech-provider keys or live voice configuration.
Use a separately installed Codex client:

```sh
"$BLEEP" mcp-config codex https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE" \
  > /absolute/bleep-mcp.toml
```

Merge the generated `[mcp_servers.bleep]` table into `~/.codex/config.toml`.
Preserve unrelated settings. Keep the absolute executable/state paths and
300-second tool timeout. Use another server name if Bleep is already configured.
Restart the server after registration. See the
[official Codex MCP configuration](https://learn.chatgpt.com/docs/extend/mcp?surface=cli).

Check `codex mcp list` and `/mcp` inside Codex. First ask it to use `bleep_status`.
Passive status and tool discovery do not start the installation host. After
receiving approval, ask for `bleep_inbox`, then read an exact item identity.
The first content operation starts the configured SDK host. Only one CLI, MCP
or background host can own an installation at a time.

Other local stdio clients can launch:

```sh
"$BLEEP" mcp https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE"
```

`mcp-config json` emits a generic launch descriptor, not every client's schema.
Claude Code supports local stdio registration; its Bleep live flow is not
independently verified here. Remote-HTTP-only clients cannot directly launch
this local server. The API origin is not an MCP endpoint. Bare `bleep mcp`
exposes metadata without a linked installation.

## 5. Send and read messages

The package supports twelve tools across its two configurations: `bleep_send`, `bleep_send_media`, `bleep_inbox`,
`bleep_read_text`, `bleep_read_media`, `bleep_setup_status`, `bleep_status`,
`bleep_cancel`, `bleep_listen`, `bleep_speak`, `bleep_end`, and
`bleep_voice_instructions`. Messaging-only configuration exposes eight tools and omits `bleep_listen`,
`bleep_speak`, `bleep_end` and `bleep_voice_instructions`. Voice configuration
exposes all twelve; use the optional setup below for Conversations.

`bleep_send` takes `{operation, text}`. `bleep_send_media` takes an operation and
absolute local `photos` and/or `voice_note` paths. Up to four JPEG/PNG photos are
supported. Text can accompany photos but cannot accompany a voice note. AAC/M4A
voice notes support up to 30 minutes and 16 MiB under the accepted signed policy.
Older messages retain their original signed limits.

Use a fresh UUID for a new message. Retry an uncertain send with the same UUID
and byte-identical inputs. A new operation after uncertainty can duplicate it.

Read with all four exact Inbox identity fields: `threadId`, `threadRevision`,
`contentId`, `envelopeDigest`. `bleep_read_media` also requires
`output_directory`: an existing absolute owner-only directory with no symlink
components. On macOS use a canonical path such as `/private/tmp/...`, not the
`/tmp` symlink. Native validation fully decodes media before returning private
exports. The caller retains or deletes these plaintext exports. Stored voice
notes are not automatically transcribed or interpreted.

Idle sync waits for authorized change hints and applies a complete covered
Inbox. Hints do not grant permission or download every attachment. Waiting
invokes no model. Explicit content reads keep fresh permission checks and are
busy during an active Conversation.

## 6. Optional live voice

Obtain a trusted public bridge configuration from the deployment operator or an
existing accepted installation. It contains relay ID, IP:port, TLS server name,
issuer and public verification keys. Raw API `/v1/config` is not this bridge
file. Guided relay setup is not shipped. Include no credentials or old test
fields `replyPcm` / `replyDelayMs` in it.

Use separate owner-only provider environment files (mode 0600, at most 16 KiB
each), outside the package. They contain their matching `OPENAI_API_KEY` or
`ELEVENLABS_API_KEY` assignments. Do not put secret values in arguments, public
configuration or logs. Use an ElevenLabs voice ID available to your account.
Speech providers can charge for conversion.

For a terminal Conversation:

```sh
"$BLEEP" conversation https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE" \
  /absolute/public-config.json bundled YOUR_VOICE_ID \
  --openai-env-file /absolute/openai.env \
  --elevenlabs-env-file /absolute/elevenlabs.env \
  --mp3-bitrate 128
```

Connect on the Phone. Type `listen`, send a short Phone turn, then type
`speak Hello from Bleep.` after its transcript appears. Type `quit` to shut down.
The CLI supplies no reasoning. Transmission acceptance does not prove hearing.

For voice-enabled Codex, replace the messaging table with a generated fragment:

```sh
"$BLEEP" mcp-config codex https://api.bleep.talk @YOUR_HANDLE my-agent "$STATE" \
  /absolute/public-config.json bundled YOUR_VOICE_ID \
  --openai-env-file /absolute/openai.env \
  --elevenlabs-env-file /absolute/elevenlabs.env \
  --mp3-bitrate 128 > /absolute/bleep-mcp.toml
```

Do not start a second owner for the same installation. Deliberate listen/speak
is supported. Automatic steering into an arbitrary existing desktop chat is
not shipped. A separately configured teacher/custom agent owns its prompts,
model, memory and wake lifecycle; none is bundled here.

## Distribution

First-party binaries are free to use, including commercially, and can be
redistributed unchanged under the attached Bleep Binary License. First-party
source remains private. Third-party rights are unchanged. The signed/notarized
archive includes notices and corresponding FFmpeg/Opus sources. Windows, Linux
and Intel Mac packages are not provided.

## Move from the old personal tap

Close the running Bleep host. Keep pairing and provider files outside the Cellar.
The repository transfer preserves old download redirects, but it does not rename
an installed local tap. Use these verified steps:

```sh
brew uninstall MauScheff/bleep/bleep
brew untap MauScheff/bleep
brew install bleep-talk/bleep/bleep
```

Restart the host with the same external state and stable Homebrew command.
The uninstall removes package files; it preserves external pairing and provider files.
