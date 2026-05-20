---
name: element-notification-sound
description: Change Element Desktop notification sound; for Matrix client notification audio not applying or needing replacement
---

# Element Desktop notification sound

**TL;DR:** Set it via Matrix `account_data` (`im.vector.web.settings.notificationSound`), not by editing `webapp.asar`.

## Required inputs

| Variable | Meaning |
|---|---|
| `$ACCESS_TOKEN` | Matrix access token for the target user |
| `$HOMESERVER` | Homeserver base URL, e.g. `https://matrix.example.com` |
| `$USER_ID` | Full Matrix ID, e.g. `@alice:example.com` |
| `$SOUND_FILE` | Path to the replacement audio file (mp3 recommended) |

If you're an AI agent and these aren't bound in the current context, ask the user or load their private skill that resolves them — don't guess.

## Symptom

Want to change Element Desktop's notification sound, or the sound isn't applying. The obvious-looking fix — replacing `media/message.ogg` and `media/message.mp3` inside `Element.app/Contents/Resources/webapp.asar` — fails on macOS:

```
cp: …/webapp.asar: Operation not permitted
```

Even with `sudo`.

## Cause

macOS Sequoia's App Management protection puts a `com.apple.provenance` xattr on `/Applications/*.app` bundles. Writes into the bundle are blocked at the kernel level regardless of UID.

Element actually reads the notification sound from a Matrix `account_data` event, not from the bundle. The bundled `.ogg`/`.mp3` are only the fallback when no `notificationSound` is set in account data.

## Fix

Upload the new sound as a Matrix media object, then PUT it into `account_data` of type `im.vector.web.settings`, field `notificationSound`. Element picks it up via `/sync`.

```bash
# URL-encode the user ID: @alice:example.com -> %40alice%3Aexample.com
USER_ID_ENC=$(printf %s "$USER_ID" | jq -sRr @uri)

# 1. Upload media -> mxc:// URL
MXC=$(curl -s -X POST "$HOMESERVER/_matrix/media/v3/upload?filename=$(basename "$SOUND_FILE")" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: audio/mpeg" \
  --data-binary @"$SOUND_FILE" | jq -r .content_uri)

# 2. GET existing account_data (must preserve sibling settings)
CUR=$(curl -s "$HOMESERVER/_matrix/client/v3/user/$USER_ID_ENC/account_data/im.vector.web.settings" \
  -H "Authorization: Bearer $ACCESS_TOKEN")

# 3. Merge notificationSound and PUT back
SIZE=$(stat -f %z "$SOUND_FILE" 2>/dev/null || stat -c %s "$SOUND_FILE")
NEW=$(echo "$CUR" | jq --arg url "$MXC" --argjson size "$SIZE" --arg name "$(basename "$SOUND_FILE")" \
  '.notificationSound = {"name":$name,"size":$size,"type":"audio/mpeg","url":$url}')

curl -s -X PUT "$HOMESERVER/_matrix/client/v3/user/$USER_ID_ENC/account_data/im.vector.web.settings" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d "$NEW"
```

PUT returns `{}` on success. No Element restart needed; it picks up on next `/sync`.

### Converting macOS system sounds (AIFF → mp3)

```bash
ffmpeg -i /System/Library/Sounds/Ping.aiff -c:a libmp3lame -q:a 2 ping.mp3
```

Element accepts `audio/mpeg`. Other codecs may work but mp3 is the reliably-supported path.

## Notes

- The GET-merge-PUT pattern is **mandatory**. PUT-ing a bare `{"notificationSound": …}` overwrites siblings in `im.vector.web.settings` (theme, language, etc.).
- If Apple loosens App Management or you ship Element from a non-`/Applications/` path, the `webapp.asar` route may work again — but `account_data` is still preferable: per-user, survives reinstalls, replicates across clients on the same account.
- Verified against Element Desktop on macOS Sequoia with a Tuwunel homeserver. The mechanism is standard Matrix client-server API and should work against any homeserver implementation.
