# ipchat — Location-Based Anonymous Chat

> Anonymous, location-based chat. Connect with anyone sharing your IP address. No signup, no servers. Messages auto-expire in 24 hours.

A single-file HTML application that turns your public IP address into a chat room. Everyone behind the same IP (home Wi-Fi, office network, café hotspot, dorm LAN) lands in the same room automatically — no invite links, no accounts, no central server storing your data.

---

## 1. Overview

`ipchat` is a zero-backend chat application built as a single self-contained HTML file. It demonstrates a serverless messaging architecture by combining three transport layers:

1. **BroadcastChannel API** — instant, free cross-tab messaging within the same browser.
2. **localStorage** — persistent 24-hour message history and presence state per room.
3. **Optional public WebSocket relay** — best-effort cross-device delivery for users on the same IP but different browsers/devices.

The room key is derived from the visitor's public IP address (fetched from `ipify`), so two people sharing an internet connection automatically end up in the same logical room without any manual pairing step.

### Design Principles

| Principle | How it's enforced |
|---|---|
| **No signup** | Nickname is auto-generated client-side (`Adjective + Animal`, e.g. `WiseSerpent`) and stored only in `localStorage`. |
| **No servers** | All transport is browser-native (`BroadcastChannel`) or third-party public relays (`socketsbay.com`). |
| **No tracking** | No cookies, no analytics, no third-party trackers. `localStorage` keys are namespaced and clearable. |
| **Ephemeral** | Messages carry a 24-hour TTL and are purged on every read/write. |
| **Progressive enhancement** | Works fully offline (same-browser) until the user opts into cross-device relay. |

---

## 2. Core Architecture

### 2.1 Transport Stack

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (ipchat)                      │
│                                                              │
│   ┌──────────────┐   ┌──────────────┐   ┌───────────────┐  │
│   │ Composer UI  │──▶│  Message Bus  │──▶│  Renderer UI   │  │
│   └──────────────┘   └──────┬───────┘   └───────────────┘  │
│                              │                              │
│         ┌────────────────────┼────────────────────┐         │
│         ▼                    ▼                    ▼         │
│  ┌────────────┐    ┌──────────────────┐   ┌──────────────┐ │
│  │ Broadcast   │    │   localStorage   │   │  WebSocket   │ │
│  │  Channel    │    │ (persistence)    │   │   (relay)    │ │
│  │ (cross-tab) │    │                  │   │ (cross-dev)  │ │
│  └────────────┘    └──────────────────┘   └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Room Resolution

Room identity is derived deterministically from the user's public IP:

```javascript
async function detectIP() {
  // 1. Try ipify primary endpoint (IPv4)
  // 2. Fallback to api64.ipify.org (IPv4 + IPv6)
  // 3. Final fallback: SHA-256 hash of browser fingerprint
  //    (userAgent + language + screen + timezone + CPU cores)
}

function roomFromIp(ip) {
  return 'ipchat-' + ip.replace(/[^a-zA-Z0-9]/g, '-').slice(0, 40);
}
```

The fingerprint fallback ensures the app still works behind restrictive networks that block external IP APIs — in that case each browser gets a unique pseudo-room.

### 2.3 Message Protocol

All messages are plain JSON objects passed through `BroadcastChannel.postMessage` or `WebSocket.send`. The wire format is intentionally minimal:

| `type` | Direction | Payload | Purpose |
|---|---|---|---|
| `hello` | out | `userId, nickname` | Announce arrival on join |
| `presence` | out | `userId, nickname` | Heartbeat every 15s |
| `bye` | out | `userId, nickname` | Announce departure on unload |
| `message` | out | `id, userId, nickname, color, text, kind, ts, reactions` | Chat message |
| `reaction` | out | `msgId, userId, emoji, ts` | Emoji reaction toggle |
| `delete` | out | `msgId, userId` | Soft-delete a message |
| `rename` | out | `userId, nickname` | Nickname change broadcast |
| `topic` | out | `topic` | Set room topic (shown in header) |
| `typing` | out | `userId, nickname` | Typing indicator (throttled 1.5s) |
| `sync-request` | out | `userId` | Ask peers for presence + recent history |

Incoming messages are routed through a single `handleIncoming(data, src)` dispatcher that ignores echoes from self (`data.userId === S.myUserId`) to avoid duplicate rendering.

### 2.4 Message Lifecycle

1. User types in composer → message object built locally with `uuid()` and `Date.now()`.
2. `addMessage(msg, mine=true)` renders immediately (optimistic UI) and persists to `localStorage`.
3. `bcSend` + `wsSend` fan out to peers.
4. Remote peers receive, dedupe by `msg.id`, render, persist.
5. Every render purges messages older than `MSG_TTL = 24h`.
6. Soft-delete replaces bubble content with `🗑️ message deleted`.

---

## 3. Feature Set

### 3.1 Messaging

| Feature | Implementation |
|---|---|
| **Plain text** | Composer textarea with auto-grow (max 160px height) |
| **Markdown subset** | ```` ``` ``` code blocks, `` `inline` ``, `**bold**`, `*italic*`, auto-linked URLs, `@mentions` highlighted |
| **Slash commands** | `/help`, `/me <action>`, `/nick <name>`, `/who`, `/clear`, `/theme dark\|light\|auto`, `/topic <text>` |
| **Mentions** | Typing `@` triggers a popup of matching online users; arrow-key navigation, Enter/Tab to insert |
| **Emoji picker** | 64-emoji grid popup, click to insert at cursor |
| **Reactions** | 7 quick reactions (👍 ❤️ 😂 🔥 🎉 👀 🙏) per message; toggle behavior — same user clicking same emoji removes it |
| **Message deletion** | Only the sender can delete their own message; soft-delete with visual strike-through style |
| **24h TTL** | `purgeMessages()` runs on every add/load; cutoff = `Date.now() - 24*60*60*1000` |
| **History sync** | On `sync-request`, peers reply with last 30 non-expired messages |
| **Local persistence** | Up to 200 most recent messages per room stored in `localStorage` under `ipchat.v1.messages.<room>` |
| **Export** | Settings → Export downloads a JSON snapshot (room, IP, messages, online list) |

### 3.2 Presence & Identity

- **Auto-generated nicknames** — `Adjective + Animal` from curated word lists (e.g. `CosmicFox`, `MistyOwl`). User can override via modal or `/nick`.
- **Stable user ID** — `crypto.randomUUID()` stored in `localStorage`; survives reloads.
- **Color-coded avatars** — HSL hue derived from `hash(nickname) % 360`, saturation 65%, lightness 55%. Consistent across all peers.
- **Online strip** — horizontal scrollable chip list with avatar + nickname, "you" badge on self.
- **Heartbeat** — every 15 seconds each client emits a `presence` message.
- **Stale detection** — peers not seen for 45 seconds are pruned from the online map.
- **Join/leave events** — `hello` on connect, `bye` on `beforeunload`. System messages can be toggled off in settings.

### 3.3 Typing Indicators

- Send: throttled to once per 1.5 seconds while typing.
- Display: animated three-dot bounce; shows up to 3 names, then `+N` overflow.
- Auto-clear after 4 seconds of inactivity.

### 3.4 Notifications

| Channel | Trigger | Toggle |
|---|---|---|
| **Tab title counter** | New message while tab unfocused | Always on |
| **Sound** | New incoming message (oscillator 660 Hz sine, 200ms decay) | Settings → Sound notifications |
| **Browser push** | New message while tab in background | Settings → Browser notifications (requests `Notification.permission`) |
| **In-app toast** | First-time room join, command results, errors | Always on |

### 3.5 Theming

Three-way theme switcher persisted across sessions:

- **🌙 Dark** — `#0a0a0f` background, `#e8eaf2` text, purple-teal gradient accents
- **☀️ Light** — `#f5f6fa` background, `#1a1c2e` text, muted accent tones
- **💻 Auto** — follows `prefers-color-scheme` media query

Theme also updates the `<meta name="theme-color">` tag so mobile browser chrome matches.

### 3.6 Settings (all stored locally)

| Setting | Default | Effect |
|---|---|---|
| Theme | `auto` | See above |
| Sound notifications | on | Plays tone on incoming message |
| Browser notifications | off | Push notifications when tab hidden |
| Enter to send | on | `Enter` sends; off → `Ctrl/Cmd+Enter` sends |
| Show system messages | on | Toggle join/leave/rename events |
| Cross-device relay | off | Connect to public WebSocket relay |
| Custom relay URL | (blank) | Override default `wss://socketsbay.com/wss2/<room>`; `<room>` placeholder auto-substituted |
| Clear local data | — | Wipes all `ipchat.v1.*` keys and reloads |
| Export chat | — | Downloads JSON snapshot |

---

## 4. UI / UX Details

### 4.1 Layout

A 5-row CSS grid inside `#app`:

```
┌──────────────────────────────────────────────┐
│ Header: logo, room pill, settings button     │
├──────────────────────────────────────────────┤
│ Online strip (horizontal scroll)             │
├──────────────────────────────────────────────┤
│                                                │
│  Messages list (scrollable, gradient-masked)   │
│                                                │
├──────────────────────────────────────────────┤
│ Typing indicator                              │
├──────────────────────────────────────────────┤
│ Composer: emoji · textarea · send              │
└──────────────────────────────────────────────┘
```

- **Glassmorphism** throughout: `backdrop-filter: blur(20px) saturate(180%)` over semi-transparent backgrounds.
- **Ambient gradient backdrop** — three radial gradients (purple top-left, teal bottom-right, purple center) fixed behind content.
- **Safe-area insets** respected for iPhone notch / home indicator via `env(safe-area-inset-*)`.
- **Max width 1200px** centered for desktop; full-width on mobile.

### 4.2 Message Bubbles

- Inbound: solid `--bg-2` background with subtle border.
- Outbound (`me`): purple-to-teal gradient tint with accent border, right-aligned, avatar on right.
- System messages: centered dashed pill, smaller font, muted color.
- Hover reveals action buttons (delete for sender, reactions for others).
- Animations: 250ms slide-in on insert, smooth scroll-to-bottom.

### 4.3 Date Dividers

Auto-inserted when the date changes between consecutive messages. Format: `Today` / `Yesterday` / `Mon, DD`.

### 4.4 "New Messages" Bar

When the user has scrolled up and a new message arrives, a floating gradient pill appears at the bottom-right. Clicking scrolls to the latest message and dismisses the bar.

### 4.5 Boot Screen

Full-screen spinner with status text that updates through stages: `Resolving your location…` → `Detecting your IP…` → `Joining room…` → fades out after 400ms.

### 4.6 Responsive Breakpoints

| Width | Behavior |
|---|---|
| > 720px | Full layout, max-width 1200px centered |
| ≤ 720px | Tighter padding, smaller avatars, room pill truncated |
| ≤ 380px | Room pill hidden entirely to save horizontal space |

---

## 5. Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Enter` | Send message (when "Enter to send" is on) |
| `Shift + Enter` | Insert newline |
| `Ctrl/Cmd + Enter` | Send (when "Enter to send" is off) |
| `/` | Begin a slash command |
| `@` | Begin a mention |
| `↑` / `↓` | Navigate mention popup |
| `Tab` / `Enter` | Accept highlighted mention |
| `Esc` | Close mention popup / emoji picker / modal |
| `⌘ ,` | Open settings (advertised in tooltip) |

---

## 6. Security & Privacy Model

### 6.1 What is Stored Locally

All under the `ipchat.v1.` namespace in `localStorage`:

| Key | Value |
|---|---|
| `nickname` | Current nickname string |
| `userId` | Persistent UUID |
| `settings` | JSON object of all settings |
| `messages.<room>` | Array of last 200 messages |
| `nickname-prompted` | Boolean — whether first-run nick modal was shown |

`Settings → Clear local data` removes every key matching `ipchat.v1.*` in one click.

### 6.2 What is Transmitted

- **To ipify**: GET request to fetch your public IP. No headers, no body, no cookies.
- **To BroadcastChannel**: full message payloads — but these never leave the browser process.
- **To WebSocket relay (if enabled)**: full message payloads in plaintext JSON to the configured relay URL. The default relay (`socketsbay.com`) is a free public service — **do not send sensitive content** when relay is on.

### 6.3 Trust Boundaries

| Layer | Trusted? | Reason |
|---|---|---|
| `localStorage` | Local only | Stays on this device |
| `BroadcastChannel` | Local only | Same browser, different tabs |
| ipify API | Trusted for IP only | Only receives your IP string |
| Public relay | **Untrusted** | Third-party service, plaintext, no auth |
| Other peers | **Untrusted** | Anyone can claim any nickname; no auth |

There is **no authentication, no nickname reservation, no message signing**. This is intentional — the app is for casual same-network chat, not for sensitive communication.

### 6.4 Input Sanitization

- All user-supplied text is passed through `escapeHtml()` before being inserted into the DOM via `innerHTML`.
- Markdown renderer escapes first, then re-applies formatting tags — so `<script>` tags in messages render as inert text.
- Links get `target="_blank" rel="noopener"`.

---

## 7. Technical Implementation Notes

### 7.1 Stack

- **Vanilla JS** — no framework, no build step, no dependencies. Runs from a single `.html` file.
- **CSS custom properties** for theming via `:root[data-theme="..."]` selectors.
- **CSS Grid + Flexbox** for layout.
- **Web APIs**: `BroadcastChannel`, `WebSocket`, `localStorage`, `crypto.subtle`, `crypto.randomUUID`, `AudioContext`, `Notification`, `BroadcastChannel`, `navigator.userAgent`.

### 7.2 Performance Choices

- Message rendering is **append-only** — new messages are inserted as individual DOM nodes rather than re-rendering the whole list.
- `renderAll()` is only called on initial load or `/clear`.
- Re-rendering of online strip and typing indicator is debounced by the 15s presence loop.
- `localStorage` writes are throttled to one per message add — no batching, but cheap enough at this scale.
- Reaction updates patch the DOM in place rather than re-rendering the parent message.

### 7.3 Resilience

| Failure mode | Behavior |
|---|---|
| ipify unreachable | Falls back to `api64.ipify.org`, then to fingerprint hash |
| WebSocket relay drops | Auto-reconnects every 3 seconds while `relay` setting is on |
| `BroadcastChannel` unsupported | `S.bc = null`, falls back to single-tab mode silently |
| `localStorage` quota exceeded | Writes wrapped in try/catch, app keeps running |
| Stale peer (no heartbeat 45s) | Pruned from online map on next presence tick |
| Tab backgrounded | Sound + browser notifications fire; tab title shows unread count |

### 7.4 Browser Compatibility

Targets modern evergreen browsers (Chrome 90+, Firefox 90+, Safari 15+). Key APIs required:

- `BroadcastChannel` (Safari 15.4+)
- `crypto.randomUUID` (with fallback to manual UUID)
- `backdrop-filter` (Safari 9+, graceful degradation)
- CSS `env(safe-area-inset-*)` (iOS 11.2+)
- `100dvh` viewport unit (modern mobile browsers)

---

## 8. Use Cases

### 8.1 Home Network

Family members on the same Wi-Fi can chat without installing anything or sharing links. Open the URL — you're in the same room.

### 8.2 Office LAN

Coworkers on the same corporate network get a shared ephemeral channel. No IT approval needed since there's no server to provision. Messages vanish in 24h, leaving no audit trail on disk.

### 8.3 Café / Public Hotspot

Anyone connected to the same café Wi-Fi can drop into the room. Useful for impromptu coordination ("who's laptop is this?") without exchanging contact info.

### 8.4 Development / Testing

Useful as a zero-config demo of BroadcastChannel + WebSocket patterns for developers learning real-time web protocols.

### 8.5 Dorm / Shared Housing

Roommates on a shared router get a persistent group chat without needing WhatsApp/Discord groups.

---

## 9. Limitations & Trade-offs

| Limitation | Reason | Mitigation |
|---|---|---|
| Same-IP requirement | Room key derives from public IP | Use relay mode + share room URL manually (not implemented) |
| No cross-network messaging | By design — privacy-first | N/A |
| No message search | Ephemeral by design | Export to JSON for archival |
| No media / file sharing | Out of scope for v1 | Could be added via relay-side storage |
| No nickname ownership | No auth layer | Acceptable for casual use |
| Relay is plaintext | Public free relay | Use custom relay URL with TLS + auth |
| IPv6 dual-stack ambiguity | `ipify` may return either v4 or v6 | Some users on dual-stack may land in different rooms |
| Carrier-grade NAT | Users behind CGNAT share an IP | Could cause unintended room sharing — feature, not bug, for same-network chat |

---

## 10. Extension Points

The codebase is structured to make several extensions straightforward:

### 10.1 Custom Relay

Point at your own WebSocket server by filling in **Settings → Custom relay URL**. Use the `<room>` placeholder to keep room isolation:

```
wss://your-relay.example.com/chat/<room>
```

The relay only needs to broadcast incoming JSON to all connected clients — no message inspection required.

### 10.2 End-to-End Encryption

The message bus is agnostic to payload encoding. A future version could:

1. Exchange ECDH public keys on `hello`.
2. Derive a shared room key per peer pair.
3. Encrypt `msg.text` before `bcSend`/`wsSend`.
4. Decrypt on receive.

The renderer would need minimal changes — just decrypt before `renderMarkdown()`.

### 10.3 File / Image Sharing

The wire protocol already supports arbitrary `kind` fields. Adding `kind: 'image'` with a base64 payload (capped at relay message size limits) would require:

- Composer: file input + drag-drop handler.
- Renderer: `<img>` branch in `renderMessage()`.
- Persistence: optional — large media could be excluded from `localStorage`.

### 10.4 Persistent Rooms

Replacing `roomFromIp(ip)` with a hash of a user-provided room name would enable invite-based rooms independent of IP, while keeping the rest of the stack intact.

---

## 11. File Structure

The entire application ships as a single HTML file (`<500` lines of CSS, `~700` lines of JS) with no external dependencies beyond the optional ipify and relay endpoints. Sections in the file:

```
<head>
  ├─ Meta tags (viewport, theme-color, PWA capabilities, SVG favicon)
  ├─ <title>
  └─ <style> (reset, tokens, layout, components, responsive, boot)
<body>
  ├─ #boot (loading overlay)
  ├─ #toasts (notification stack)
  ├─ #app
  │   ├─ <header> (logo, room pill, settings)
  │   ├─ .online-strip (presence chips)
  │   ├─ .messages (scrollable feed)
  │   ├─ .new-bar (jump-to-bottom pill)
  │   ├─ .typing (indicator)
  │   └─ .composer (emoji + textarea + send)
  ├─ #nickModal (first-run nickname prompt)
  ├─ #settingsModal (full settings panel)
  └─ <script>
      ├─ Constants (EMOJIS, REACTIONS, TTLs)
      ├─ State object S
      ├─ Utilities ($, uuid, colorFor, fmtTime, escapeHtml)
      ├─ Persistence (loadLS/saveLS/removeLS)
      ├─ IP detection (detectIP, fingerprint, roomFromIp)
      ├─ Transport (setupBC, setupWS, handleIncoming)
      ├─ Outgoing (broadcastMessage, sendTyping, sendReaction, ...)
      ├─ Rendering (renderAll, renderMessage, renderOnline, renderTyping)
      ├─ Composer logic (mentions, emoji picker, shortcuts)
      ├─ Commands (/help, /me, /nick, /who, /clear, /theme, /topic)
      ├─ Settings UI (theme, switches, relay URL)
      ├─ Nickname modal
      ├─ Sound + browser notifications
      ├─ Presence loop (heartbeat, beforeunload, visibility)
      └─ init() entry point
```

---

## 12. Quick Start

1. Save the HTML file to disk (or host on any static web server — GitHub Pages, Netlify, Cloudflare Pages, etc.).
2. Open the URL in any modern browser.
3. The boot screen will resolve your IP, derive your room, and connect.
4. (Optional) Click the ⚙️ icon → toggle **Cross-device relay** to enable cross-browser messaging on your network.
5. (Optional) Set a custom nickname via the prompt or `/nick <name>`.
6. Start typing. `Enter` sends.

That's it. No accounts, no servers, no setup.
