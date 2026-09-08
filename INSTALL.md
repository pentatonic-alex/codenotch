# Installing this Codenotch build

This is a **fork** of [vinzdg/codenotch](https://github.com/vinzdg/codenotch)
with one fix and one build tweak, for sharing internally until the fix lands
upstream. MIT licensed; all credit to the original author.

**What's different from upstream:**
- **Claude usage keychain fix** — the default Claude profile now reads the
  SHA-256-suffixed keychain service, not just the bare name, so the Claude ring
  actually shows a reading when `CLAUDE_CONFIG_DIR` is set. (Upstream PR:
  [vinzdg/codenotch#25](https://github.com/vinzdg/codenotch/pull/25).)
- **Deployment target lowered to macOS 15** so it runs on Sequoia as well as
  Tahoe. (Upstream targets macOS 26.)

## Install (from the DMG on the Releases page)

The DMG is **ad-hoc signed, not notarized**, so macOS Gatekeeper will warn.
That's expected for an internal build. To install:

1. Open the `.dmg` and **drag `Codenotch.app` onto the `Applications` folder**.
2. First launch: in `/Applications`, **right-click `Codenotch` → Open → Open**.
3. If macOS still refuses ("damaged" / "cannot be opened"), clear quarantine
   once in Terminal, then open it:
   ```sh
   xattr -dr com.apple.quarantine /Applications/Codenotch.app
   open /Applications/Codenotch.app
   ```
4. On first read, macOS asks to use the **`Claude Code-credentials`** keychain
   item — click **Always Allow**. (Codenotch only ever *reads* these tokens to
   call each tool's own usage API — see the security note below.)

It is a **menu-bar app**: the readings appear pinned to a **screen edge / the
notch**, not in the Dock.

> If you rebuild it yourself, the ad-hoc identity changes, so you'll have to
> re-grant the keychain "Always Allow" once more.

## Build it yourself instead

Requires full **Xcode** (not just Command Line Tools) and `xcodegen`:

```sh
brew install xcodegen
make build DEV_SIGN='CODE_SIGN_IDENTITY=- DEVELOPMENT_TEAM= CODE_SIGN_STYLE=Automatic CODE_SIGNING_REQUIRED=NO'
```

Then copy the built `Codenotch.app` from `DerivedData/.../Debug/` into
`/Applications`.

### Keychain prompt keeps coming back?

A self-built (ad-hoc) app has no stable code identity, so macOS re-asks for the
keychain token on every launch even after you click **Always Allow**. Sign it
once with a stable self-signed identity to make the grant stick:

```sh
Scripts/sign-local.sh   # signs /Applications/Codenotch.app
```

No Apple Developer account needed. Grant the prompt one more time after signing;
it won't ask again. (Not needed if you install from the DMG above.)

## What it reads (security)

Codenotch reads each installed tool's own auth (Claude Code, Cursor, Codex,
Grok, OpenCode, etc.) **only to call that tool's own usage API**, over HTTPS to
that vendor's own host. It does not write, log, or send credentials anywhere
third-party, and there is no telemetry. It runs unsandboxed because reading
another app's keychain item requires it; macOS still gates each read behind the
per-item "Always Allow" prompt.
