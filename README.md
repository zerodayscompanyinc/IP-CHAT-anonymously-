# ipchat — Complete Guide & Documentation

> **Anonymous, location-based, globally-connected chat.** Connect with anyone sharing your IP — or anyone worldwide via WebRTC + MQTT. No signup, no servers, no API keys. Messages auto-expire in 24 hours.

A single-file HTML chat application that works on any device, in any of 195 countries, with zero setup. Open the URL, pick a nickname, and start chatting — locally on your network, or globally with anyone on the planet.

---

# link URL here:
https://ipchnonymous.vercel.app/

---

## Table of Contents

1. [Quick Start — 30 seconds to your first message](#1-quick-start--30-seconds-to-your-first-message)
2. [Tutorial — Walk through every feature](#2-tutorial--walk-through-every-feature)
3. [User Guide — How to use each part of the app](#3-user-guide--how-to-use-each-part-of-the-app)
4. [Architecture Overview](#4-architecture-overview)
5. [Feature Reference](#5-feature-reference)
6. [Keyboard Shortcuts & Commands](#6-keyboard-shortcuts--commands)
7. [Security & Privacy](#7-security--privacy)
8. [Technical Implementation](#8-technical-implementation)
9. [Use Cases](#9-use-cases)
10. [Limitations & Troubleshooting](#10-limitations--troubleshooting)
11. [Extending ipchat](#11-extending-ipchat)
12. [Appendix](#12-appendix)

---

## 1. Quick Start — 30 seconds to your first message

### Five-step quickstart

1. **Open the URL.** Save the HTML file anywhere (USB stick, GitHub Pages, Netlify, your own server, even a local `file://` open) and open it in any modern browser.
2. **Wait for the boot screen.** A spinner appears for ~1 second while ipchat detects your public IP and loads global transports.
3. **Pick a nickname.** A modal pops up the first time. Accept the suggested random name (e.g. `WiseSerpent`), type your own, or click 🎲 Random to roll a new one. Click **Save**.
4. **Type a message.** Click into the bottom text box and type `Hello world!`. Press `Enter` to send.
5. **Done.** Your message appears in the room. Anyone else on your IP — or anyone worldwide with global mode on — will see it within seconds.

### What happens automatically

- **IP detection** via ipify (with fallbacks).
- **Country detection** from browser language or timezone — no API call.
- **BroadcastChannel** for instant cross-tab sync on the same browser.
- **WebRTC (PeerJS)** loads in the background — connects you to peers worldwide.
- **MQTT (EMQX public broker)** loads in parallel as a fallback pub/sub channel.
- **Status badge** in the header (`global · P2P+MQTT`) shows what transports are live.

---

## 2. Tutorial — Walk through every feature

This tutorial covers every interactive surface of the app, in the order you'd naturally encounter them.

### Tutorial 2.1 — The header

The header at the top of the screen contains:

```
[💬 ipchat] [🟢 Room: ipchat-203-0-113-50] [🌍 global · P2P+MQTT] [🔑] [🌐] [🔗] [⚙️]
```

| Element | What it does |
|---|---|
| 💬 **ipchat logo** | Visual branding only — not clickable. |
| **🟢 Room pill** (clickable) | Shows your current room key and connection status. Click to open the Join Room modal. |
| **🌍 Global badge** | Shows live transport status: `global · P2P+MQTT`, `global · P2P`, `local only`, or `connecting…`. |
| **🔑 Join button** | Open the manual Join Room modal (enter any IP+port). |
| **🌐 Public button** | Open the Public Rooms directory (browse / create / manage). |
| **🔗 Share button** | Open the Share modal (link, QR code, native share). |
| **⚙️ Settings button** | Open the Settings panel. |

**Try it:** Click the room pill right now. A modal appears letting you join any other room by IP+port. Cancel for now.

### Tutorial 2.2 — The online strip

Just below the header, a horizontal strip shows everyone currently in the room:

```
[online] [🇵🇭 avatar WiseSerpent (you)] [🇺🇸 avatar CosmicFox] [🇯🇵 avatar MistyOwl] [🇵🇭🇺🇸🇯🇵 3 online · 3 countries]
```

- Each user shows **their country flag** (auto-detected), avatar (colored with initials), and nickname.
- Yourself is marked with `(you)` and a highlighted border.
- The right side shows the **online count + countries count** with up to 5 flags.

**Try it:** Open the page in two browser tabs. Within 15 seconds (presence heartbeat), both tabs will see each other in the online strip.

### Tutorial 2.3 — Sending messages

Type in the bottom text box. As you type:

- The text box **auto-grows** up to 160px tall, then scrolls internally.
- A **typing indicator** appears at the bottom of the message list ("CosmicFox is typing…").
- Press `Enter` to send. `Shift+Enter` for a newline.

After sending:

- Your message appears right-aligned with a purple-to-teal gradient bubble.
- It includes your nickname, timestamp, and avatar.
- It broadcasts on every active transport (BroadcastChannel, WebRTC, MQTT, optional WebSocket relay).

**Try it:** Type `/me waves hello` and press Enter. Your action appears in italics in the chat.

### Tutorial 2.4 — Markdown & formatting

Your messages support a markdown subset:

| You type | What renders |
|---|---|
| `` `inline code` `` | `inline code` (monospace, dark background) |
| ` ```code block``` ` | Multi-line code block |
| `**bold**` | **bold** |
| `*italic*` | *italic* |
| `https://example.com` | Auto-linked clickable URL |
| `@CosmicFox` | Highlighted mention |
| Line breaks | Rendered as `<br>` |

**Try it:** Send this message:
```
Hey @WiseSerpent — check out **this link**: https://example.com
```

### Tutorial 2.5 — Mentions

Typing `@` triggers a popup of matching online users. Navigate with arrow keys, accept with `Enter` or `Tab`, dismiss with `Esc`.

**Try it:**
1. Open two tabs and set different nicknames.
2. In tab A, type `@` — a popup appears listing users from tab B.
3. Use ↓ to highlight a user, press Enter — the mention is inserted.

### Tutorial 2.6 — Emoji picker

Click the 😊 icon next to the text box. A 64-emoji grid pops up. Click any emoji to insert it at the cursor position.

### Tutorial 2.7 — Reactions

Hover over any message (not your own). Seven quick-reaction buttons appear:

```
👍 ❤️ 😂 🔥 🎉 👀 🙏
```

Click one to react. Click the same one again to remove your reaction. Reaction counts appear below the message bubble.

**Try it:** Send a message in tab A, then react to it from tab B. The reaction appears instantly in both tabs.

### Tutorial 2.8 — Deleting messages

Hover over **your own** message. A red **delete** button appears. Click it to soft-delete — the bubble becomes italic gray with "🗑️ message deleted".

Only the sender can delete their own messages. Deletion propagates to all peers.

### Tutorial 2.9 — Joining a different room

Click 🔑 or the room pill. The Join modal appears:

1. **IP address** — accepts IPv4 (`192.168.1.10`), IPv6 (`2600:1900::f00`), or hostname (`acme-corp`).
2. **Port** — optional, 1–65535. Leave blank for no port.
3. **🎲 Random IP** — generates a `10.x.x.x` private IP + random port for a throwaway room.
4. **📡 Use my IP** — re-detects your public IP via ipify.
5. **Recent rooms** — last 8 joined rooms with one-click autofill.
6. **🚪 Join Room** — switches you to the new room instantly.

When you switch, ipchat tears down all transports, resets message history, and re-announces on the new room key.

**Try it:** Click 🎲 Random IP, then Join. You're now in a fresh empty room. Type a message — it'll be the first.

### Tutorial 2.10 — Public Rooms

Click 🌐 to open the Public Rooms modal. Three tabs:

#### Browse tab
- Search box — full-text search across name + description + category.
- Category chips — `all`, `general`, `tech`, `gaming`, `music`, `study`, `language`, `random`, `local`.
- 8 built-in rooms (🌍 Global Lobby, 💻 Tech Talk, 🎲 Random, 🎮 Gaming Lounge, 🎵 Music Room, 📚 Study Hall, 🗣️ Language Exchange, 😂 Memes).
- Each room card has a **Join** button — click to enter instantly.

#### Create Room tab
- **Room name** (required, max 32 chars).
- **Description** (optional, max 120 chars).
- **Category** — dropdown.
- **Room key** — auto-generated as `pub-<slug>-<4-char-suffix>`; can't be edited.
- Click **🚀 Create & Join Room** — added to your local public list and you're joined.

#### My Rooms tab
- Lists rooms you've created.
- Click to select; click **Remove from public list** to unlist (the room is still joinable by direct key, just no longer shown in your Browse tab).

**Try it:** Click 🌐 → Browse → click the 💻 Tech Talk card. You're now in a public tech-focused room. Anyone else who joins `pub-tech` from any device will be here.

### Tutorial 2.11 — Sharing your room

Click 🔗 to open the Share modal:

- **QR code** — 200×200 PNG, scan with any phone camera.
- **Shareable link** — `https://your-host/#room=<encoded-ip[:port]>`.
- **Copy button** — copies the link to clipboard.
- **Room key display** — the raw `ip:port` for manual entry on the Join modal.
- **⬇️ QR PNG** — download the QR as a PNG file.
- **📤 Native share** — uses `navigator.share()` (mobile), falls back to clipboard copy on desktop.

Anyone who opens the share URL will auto-join your room on page load.

**Try it:**
1. Click 🔗 in any room.
2. Click **Copy** — the link is now on your clipboard.
3. Paste it into a new browser tab. The page loads and auto-joins your room.

### Tutorial 2.12 — Settings

Click ⚙️ to open the Settings panel:

#### Appearance
Three-way theme switcher:
- 🌙 **Dark** — `#0a0a0f` background, purple-teal accents.
- ☀️ **Light** — `#f5f6fa` background, muted accents.
- 💻 **Auto** — follows OS preference.

#### Your identity
Change your nickname (max 24 chars), or click 🎲 New for a random one.

#### Switches
| Switch | Default | Effect |
|---|---|---|
| Sound notifications | on | Plays a tone on new message |
| Browser notifications | off | OS-level push notifications when tab is backgrounded |
| Enter to send | on | `Enter` sends; off → `Ctrl/Cmd+Enter` sends |
| Show system messages | on | Toggle join/leave/rename events |
| 🌍 Global mode | **on** | WebRTC + MQTT worldwide; turn off for local-only mode |

#### Live transports panel
A real-time status box showing each transport's state:
- 📡 BroadcastChannel — `on` (always, when supported)
- 🔗 WebRTC / PeerJS — `on` / `off` / `loading…` / `err`
- 📨 MQTT / EMQX broker — same states
- 🔌 Custom WebSocket relay — same states

#### Custom relay (optional)
- Toggle the WebSocket relay on/off.
- Enter a custom relay URL — use `<room>` placeholder, e.g. `wss://your-relay.com/wss/<room>`.

#### Data controls
- **🗑️ Clear local data** — wipes all `ipchat.v1.*` keys from localStorage and reloads.
- **⬇️ Export chat** — downloads JSON snapshot of room, IP, messages, online list.

### Tutorial 2.13 — Deep-link auto-join

If a URL contains `#room=<key>` in its hash, ipchat auto-joins that room on load instead of using your IP.

**Try it:** Take your share URL (from Tutorial 2.11). Open it in an incognito window. The page boots, then 800ms later auto-joins your room — no clicks needed.

This is useful for embedding room links in tweets, emails, posters, QR codes, Slack channels — anyone who clicks joins instantly.

### Tutorial 2.14 — Slash commands

Type any of these commands in the composer (they start with `/`):

| Command | Action |
|---|---|
| `/help` | Show available commands |
| `/me waves` | Send an action (italicized) |
| `/nick NewName` | Change your nickname |
| `/who` | List online users |
| `/clear` | Clear your local message view |
| `/theme dark` | Switch theme (`dark`, `light`, or `auto`) |
| `/topic My Topic` | Set the room topic (shown in header) |
| `/join 1.2.3.4:9000` | Switch to room `1.2.3.4:9000` |
| `/room` | Show current room key |
| `/global on` | Toggle global mode on/off |

**Try it:** Type `/theme light` — the app instantly switches to light mode.

### Tutorial 2.15 — Notifications

- **Tab title counter** — when the tab is backgrounded and new messages arrive, the title shows `(3) ipchat · location chat`.
- **Sound** — a 660 Hz sine wave plays on new messages (toggle in Settings).
- **Browser push** — if you enable "Browser notifications" in Settings, OS-level notifications appear even when the tab is hidden.

**Try it:** Open two tabs. In tab A, switch to a different browser window. In tab B, send a message. Tab A's title should update with the unread count.

---

## 3. User Guide — How to use each part of the app

This section is a reference-style guide for specific tasks. Jump to any subsection.

### 3.1 How to set or change your nickname

**First time:**
- A modal appears automatically on first load.
- Accept the suggested name, type your own (max 24 chars), or click 🎲 Random.
- Click **Save**.

**Later:**
- Open Settings (⚙️).
- Edit the **Your identity** field.
- Click outside the box to save.

**Via command:**
- Type `/nick NewName` in the composer and press Enter.

Your nickname is broadcast to all peers with a rename event. Old messages keep your old name (they're snapshots), but new messages show your new name.

### 3.2 How to find your current room key

Three ways:
1. Look at the **room pill** in the header — it shows `Room: ipchat-203-0-113-50` (or similar).
2. Type `/room` — a toast shows the full key.
3. Click 🔗 Share — the room key is shown in the modal.

### 3.3 How to invite someone to your room

**Same network (home Wi-Fi, office LAN):**
- Just share the URL of the ipchat page. They'll auto-land in your room because you share an IP.

**Different network (cross-country, cross-continent):**
- Click 🔗 Share → Copy the link → send via any channel (email, chat, tweet).
- Or download the QR PNG → print on a poster → people scan to join.
- Or use **📤 Native share** on mobile to send via WhatsApp/iMessage.
- Ensure both you and your friend have **🌍 Global mode ON** (default) so WebRTC + MQTT can carry messages across networks.

### 3.4 How to create a public room

1. Click 🌐 Public.
2. Switch to **+ Create Room** tab.
3. Enter a room name (e.g. `Coffee Lounge`).
4. (Optional) Enter a description.
5. Pick a category.
6. The room key is auto-generated as `pub-coffee-lounge-7k3x` — you can't edit it.
7. Click **🚀 Create & Join Room**.

The room is added to your local public list. Anyone else on the same browser sees it in their Browse tab. To make it visible to other browsers/devices, share the room key directly or use the relay-based directory sync described in §11.

### 3.5 How to switch themes

Three ways:
1. **Settings → Appearance** → click 🌙 / ☀️ / 💻.
2. **`/theme dark`** / `/theme light` / `/theme auto` command.
3. Set theme to **Auto** to follow your OS preference.

Theme is persisted across sessions in localStorage.

### 3.6 How to enable or disable global mode

Global mode controls whether ipchat uses WebRTC + MQTT to reach peers on different networks.

**To turn off (local-only mode):**
- Open Settings → toggle **🌍 Global mode** off.
- Or type `/global off`.

When off, ipchat only syncs via BroadcastChannel (same browser tabs) and any custom WebSocket relay you've configured. Useful for restrictive networks or privacy-conscious use.

**To turn on (default):**
- Toggle it on in Settings, or `/global on`.

### 3.7 How to use a custom WebSocket relay

Some users prefer to run their own relay for full control:

1. Stand up a WebSocket server that broadcasts incoming JSON to all connected clients. Examples: `websocketd`, `ws`, a small Node.js script.
2. Open Settings → toggle **Custom WebSocket relay** on.
3. Enter your relay URL in **Custom relay URL** field. Use `<room>` as a placeholder: `wss://your-relay.example.com/chat/<room>`.
4. The relay is now used in addition to WebRTC + MQTT (not instead of).

Custom relays are useful when:
- You're behind a corporate firewall that blocks WebRTC.
- You want guaranteed low-latency delivery in a specific region.
- You want full audit/control over message routing.

### 3.8 How to clear all your data

Open Settings → click **🗑️ Clear local data** → confirm.

This wipes all `ipchat.v1.*` keys from localStorage:
- Nickname, user ID, settings
- All messages from all rooms you've visited
- Recent rooms, public rooms, my rooms
- Last room joined

The page reloads with a fresh state.

### 3.9 How to export a chat

Open Settings → click **⬇️ Export chat**.

A JSON file downloads containing:
- `room` — current room key
- `ip` — current IP
- `exportedAt` — ISO timestamp
- `messages` — array of all messages (including deleted ones, marked)
- `online` — array of online users at export time

Useful for archival, sharing a conversation, or moving to a different browser.

### 3.10 How to see who's online

The **online strip** below the header shows every user in the room. The right side shows the total count and country count with flags.

To get a text list, type `/who` — a toast shows `Online (3): WiseSerpent, CosmicFox, MistyOwl`.

Presence is updated every 15 seconds via heartbeat. Users not seen for 45 seconds are pruned.

### 3.11 How to know if your message was delivered

ipchat is **best-effort** — there's no delivery confirmation mechanism. Indicators of delivery:

- **Online strip** shows the recipient as online.
- **Typing indicator** appears when they start typing back.
- **Replies** arrive in your room — that's the strongest signal.

For guaranteed delivery, you'd need a custom relay with persistence — see §11.

### 3.12 How to handle a slow connection

If messages lag or peers seem "frozen":
1. Check the **global badge** in the header — if it says `connecting…`, transports are still coming up.
2. Open Settings → **Live transports** panel — check if WebRTC and MQTT show `on`.
3. If both show `err`, your network is likely blocking WebSockets/WebRTC. Try a different network or enable a custom WebSocket relay (§3.7).
4. Refresh the page — transports re-initialize on reload.

### 3.13 How to leave a room

Just close the tab, or click 🔑 Join → enter a different room key.

A `bye` message is broadcast on `beforeunload` so other peers see you leave. If your browser crashes, the 45-second heartbeat timeout will eventually prune you from the online list.

---

## 4. Architecture Overview

### 4.1 Transport stack

ipchat uses **four parallel transports**, all running simultaneously:

```
┌────────────────────────────────────────────────────────────────────┐
│                       Browser (ipchat)                              │
│                                                                     │
│   ┌────────────┐  ┌────────────┐  ┌──────────┐  ┌──────────────┐  │
│   │ Composer   │→ │ Message    │→ │ Renderer │  │   UI: header, │  │
│   │ + Markdown │  │ Bus        │  │ + online │  │   online,    │  │
│   └────────────┘  └─────┬──────┘  └──────────┘  │   messages,   │  │
│                         │                        │   composer    │  │
│                         ▼                        └──────────────┘  │
│              ┌──────────┴───────────┐                              │
│              │   remoteSend(obj)     │  ← fan-out to all transports│
│              └──────────┬───────────┘                              │
│       ┌─────────┬──────┴───────┬─────────────┐                   │
│       ▼         ▼              ▼             ▼                    │
│  ┌────────┐ ┌────────┐  ┌────────────┐  ┌──────────┐             │
│  │ Broad- │ │ WebRTC │  │   MQTT     │  │ WebSocket│             │
│  │  cast  │ │ PeerJS │  │  EMQX      │  │  (custom)│             │
│  │Channel │ │  mesh  │  │ public     │  │  relay   │             │
│  │(local) │ │(global)│  │ broker     │  │ (optional)│             │
│  └────────┘ └────────┘  └────────────┘  └──────────┘             │
│       │           │             │                │                │
│       └─────┬─────┴──────┬──────┴────────────────┘                │
│             ▼            ▼                                         │
│      Same browser    Any device, any country                        │
│      tabs (instant)  (195 countries, no API key)                   │
└────────────────────────────────────────────────────────────────────┘
```

| Transport | Scope | Use case | Cost |
|---|---|---|---|
| **BroadcastChannel** | Same browser, different tabs | Open ipchat in 3 tabs — they sync instantly | Free, browser-native |
| **WebRTC (PeerJS)** | Any device, worldwide | Cross-network chat via P2P mesh + hub | Free, uses public PeerJS broker |
| **MQTT (EMQX)** | Any device, worldwide | Pub/sub fallback when WebRTC blocked | Free, public EMQX broker |
| **WebSocket relay** | Any device, anywhere | Optional user-supplied relay | Self-hosted or third-party |

### 4.2 Room resolution

There are **two ways to land in a room**:

#### Auto (default)
Room key is derived from your public IP:
```javascript
async function detectIP() {
  // 1. Try ipify primary endpoint (IPv4)
  // 2. Fallback to api64.ipify.org (IPv4 + IPv6)
  // 3. Final fallback: SHA-256 hash of browser fingerprint
}

function roomFromIp(ip) {
  return 'ipchat-' + ip.replace(/[^a-zA-Z0-9]/g, '-').slice(0, 40);
}
```

#### Manual join (🔑)
User enters any IP/hostname + optional port. The combined string `ip:port` is normalized through the same `roomFromIp()` function. Two users who enter the same key land in the same room — regardless of physical location.

#### Deep-link auto-join
If the URL contains `#room=<key>`, ipchat calls `joinRoom(key, ...)` on init after an 800 ms delay.

### 4.3 Country detection (no API)

Country is derived locally — no network call:

1. Try `navigator.languages` — if any ends in a region code (e.g. `en-PH`), use `PH`.
2. Try `Intl.DateTimeFormat().resolvedOptions().timeZone` and map the city name to a country code via a built-in dictionary of 60+ cities.
3. If both fail, return `null` — the user shows the 🌐 generic flag.

This is used to render flags next to each user in the online strip.

### 4.4 Message protocol

All messages are plain JSON. Every transport carries the same payload format:

| `type` | Direction | Payload | Purpose |
|---|---|---|---|
| `hello` | out | `userId, nickname, country, locale` | Announce arrival on join |
| `presence` | out | `userId, nickname, country, locale` | Heartbeat every 15s |
| `bye` | out | `userId, nickname` | Announce departure on unload |
| `message` | out | `id, userId, nickname, color, country, locale, text, kind, ts, reactions` | Chat message |
| `reaction` | out | `msgId, userId, nickname, emoji, ts` | Emoji reaction toggle |
| `delete` | out | `msgId, userId` | Soft-delete a message |
| `rename` | out | `userId, nickname` | Nickname change broadcast |
| `topic` | out | `topic` | Set room topic (header) |
| `typing` | out | `userId, nickname, country` | Typing indicator (throttled 1.5s) |
| `sync-request` | out | `userId` | Ask peers for presence + recent history |

**Cross-transport dedup:** Every message has a `uuid()` ID. The receiver maintains a `seenIds` Set (capped at 2000 entries, FIFO eviction). If the same ID arrives via WebRTC, MQTT, and WebSocket, it's only rendered once.

### 4.5 WebRTC mesh topology (PeerJS)

```
                  ┌─────────────┐
                  │  PeerJS     │
                  │  cloud broker│
                  │ 0.peerjs.com│
                  └──────┬──────┘
                         │ signaling
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
   ┌────────┐       ┌────────┐        ┌────────┐
   │ Peer A │◀─────▶│ Peer B │◀──────▶│ Peer C │
   │ (hub)  │       │(client)│        │(client)│
   └────────┘       └────────┘        └────────┘
       │                                              Star topology with
       └── re-broadcasts every message to all peers   hub failover
```

**How it works:**
1. Every peer tries to claim the hub ID `iphub-<room>` first.
2. The first peer to succeed becomes the hub; others become clients and connect to the hub.
3. Hub re-broadcasts every incoming message to all connected clients.
4. Clients ALSO maintain direct DataConnections to every peer they've seen, so messages flow even if the hub is briefly down.
5. If the hub disconnects, every remaining client tries to claim the hub ID after a random backoff (800–2300 ms). Whoever wins becomes the new hub.

**Why this design:**
- Free public PeerJS broker requires no API key.
- Star topology minimizes connection count (N instead of N²).
- Hub failover ensures no single point of failure.
- Direct peer-to-peer fallback means messages can flow during hub transitions.

### 4.6 MQTT pub/sub (EMQX)

```
   ┌──────────────────────────────────────┐
   │  EMQX public broker                  │
   │  wss://broker.emqx.io:8084/mqtt      │
   │                                      │
   │  Topic: ipchat/ipchat-203-0-113-50   │
   └────────────┬─────────────┬───────────┘
                │             │
                ▼             ▼
            ┌──────┐      ┌──────┐
            │Peer A│      │Peer B│
            │ (sub)│      │ (sub)│
            └──────┘      └──────┘
```

- Public EMQX broker at `wss://broker.emqx.io:8084/mqtt` — no API key, no signup, works in 195 countries.
- Each room subscribes to topic `ipchat/<room>`.
- `mqtt.publish(topic, JSON.stringify(msg))` to broadcast.
- All peers subscribed to the same topic receive every message.
- QoS 0 (fire-and-forget) — lowest latency, no delivery guarantees.
- Auto-reconnect with 3-second backoff.

### 4.7 Message lifecycle

1. User types in composer → message object built locally with `uuid()`, `Date.now()`, country, locale.
2. `addMessage(msg, mine=true)` renders immediately (optimistic UI) and persists to localStorage.
3. `remoteSend({type:'message', ...msg})` fans out to all four transports.
4. Remote peers receive, dedupe by `msg.id`, render, persist.
5. Every render purges messages older than 24h.
6. Soft-delete replaces bubble content with `🗑️ message deleted`.

### 4.8 File structure (single HTML file)

```
<head>
  ├─ Meta tags (viewport, theme-color, PWA, SVG favicon)
  ├─ <title>
  └─ <style> (~500 lines: reset, tokens, layout, components, public-rooms, share, responsive, boot)
<body>
  ├─ #boot (loading overlay)
  ├─ #toasts (notification stack)
  ├─ #app
  │   ├─ <header> (logo, room pill, global badge, 🔑 join, 🌐 public, 🔗 share, ⚙️ settings)
  │   ├─ .online-strip (presence chips with country flags)
  │   ├─ .messages (scrollable feed)
  │   ├─ .new-bar (jump-to-bottom pill)
  │   ├─ .typing (indicator)
  │   └─ .composer (emoji + textarea + send)
  ├─ #nickModal (first-run nickname prompt)
  ├─ #joinModal (manual IP + port room join)
  ├─ #publicModal (browse / create / my rooms — 3 tabs)
  ├─ #shareModal (QR canvas + link + key + native share)
  ├─ #settingsModal (full settings panel with live transports)
  └─ <script> (~1300 lines)
      ├─ Constants (EMOJIS, REACTIONS, TTLs, MAX_RECENT_ROOMS, PUBLIC_CATS, BUILTIN_PUBLIC_ROOMS)
      ├─ State object S (transports, online, messages, country, locale)
      ├─ Utils ($, uuid, colorFor, fmtTime, escapeHtml, isValidIp, isValidPort, genRoomKey, detectCountry, countryFlag, countryName)
      ├─ Persistence (loadLS/saveLS/removeLS, getRecentRooms, pushRecentRoom, getPublicRooms, addPublicRoom, removePublicRoom, getMyRooms)
      ├─ IP detection (detectIP, fingerprint, roomFromIp, randomIp)
      ├─ CDN loader (loadScript, loadPeerJS, loadMQTT)
      ├─ Transports:
      │   ├─ BroadcastChannel (setupBC, bcSend)
      │   ├─ WebRTC / PeerJS (setupPeerJS, tryBecomeHub, becomeClient, connectToHub, wireConn, p2pSend, closePeerJS)
      │   ├─ MQTT / EMQX (setupMQTT, mqttSend, closeMQTT)
      │   ├─ WebSocket (setupWS, wsSend, closeWS)
      │   └─ Unified fan-out: remoteSend(obj)
      ├─ Transport UI (setTransport, updateGlobalBadge)
      ├─ Incoming handler (handleIncoming with seenIds dedupe)
      ├─ Outgoing (broadcastMessage, sendTyping, sendReaction, sendBye)
      ├─ Rendering (renderAll, renderMessage, renderOnline, renderTyping, renderRecentRooms)
      ├─ Composer (mentions, emoji picker, shortcuts)
      ├─ Commands (/help, /me, /nick, /who, /clear, /theme, /topic, /join, /room, /global)
      ├─ Manual join (joinRoom, joinRoomFromInput, setupJoinModal)
      ├─ Public rooms (setupPublicModal, renderPublicCats, renderPublicGrid, renderMyRooms)
      ├─ Share (shareUrl, drawQR, setupShareModal)
      ├─ Deep-link routing (checkHashJoin)
      ├─ Settings UI (theme, switches, relay URL, global mode)
      ├─ Nickname modal
      ├─ Sound + browser notifications
      ├─ Presence loop (heartbeat, beforeunload, visibility)
      └─ init() entry point (parallel CDN loads, hash auto-join)
```

---

## 5. Feature Reference

### 5.1 Messaging

| Feature | Implementation |
|---|---|
| **Plain text** | Composer textarea with auto-grow (max 160px height) |
| **Markdown subset** | ```` ``` ``` code blocks, `` `inline` ``, `**bold**`, `*italic*`, auto-linked URLs, `@mentions` highlighted |
| **Slash commands** | `/help`, `/me <action>`, `/nick <name>`, `/who`, `/clear`, `/theme dark\|light\|auto`, `/topic <text>`, `/join <ip[:port]>`, `/room`, `/global on\|off` |
| **Mentions** | `@` triggers popup of online users; arrow-key navigation, Enter/Tab to insert |
| **Emoji picker** | 64-emoji grid popup, click to insert at cursor |
| **Reactions** | 7 quick reactions (👍 ❤️ 😂 🔥 🎉 👀 🙏) per message; toggle behavior |
| **Message deletion** | Only sender can delete; soft-delete with strike-through style |
| **24h TTL** | `purgeMessages()` runs on every add/load; cutoff = `now - 24h` |
| **History sync** | On `sync-request`, peers reply with last 30 non-expired messages |
| **Local persistence** | Up to 200 most recent messages per room in localStorage |
| **Export** | Settings → Export downloads JSON snapshot |
| **Cross-transport dedup** | `seenIds` Set (cap 2000, FIFO) prevents double-rendering |

### 5.2 Presence & Identity

- **Auto nicknames** — `Adjective + Animal` (e.g. `CosmicFox`, `MistyOwl`).
- **Stable user ID** — `crypto.randomUUID()` stored in localStorage.
- **Color-coded avatars** — HSL hue from `hash(nickname) % 360`.
- **Country flags** — auto-detected from browser language or timezone.
- **Online strip** — horizontal scrollable chip list with flag + avatar + nickname.
- **Heartbeat** — every 15 seconds.
- **Stale detection** — peers not seen for 45 seconds are pruned.
- **Join/leave events** — `hello` on connect, `bye` on `beforeunload`.

### 5.3 Global mode (WebRTC + MQTT)

**On by default.** Provides cross-network, cross-country messaging with no API key.

- **WebRTC mesh** via PeerJS — free public broker at `0.peerjs.com`. Star topology with hub failover.
- **MQTT pub/sub** via EMQX public broker at `wss://broker.emqx.io:8084/mqtt`. Topic: `ipchat/<room>`.
- **Dynamic CDN load** — `peerjs.min.js` and `mqtt.min.js` are loaded async from `unpkg.com` only when global mode is on.
- **Live transport status** — Settings → Live transports panel shows real-time state of each transport.

Turn off via Settings → 🌍 Global mode, or `/global off`.

### 5.4 Manual room join (🔑)

The Join Room modal accepts:
- IPv4 (`192.168.1.10`), IPv6 (`2600:1900::f00`), or hostname (`acme-corp`).
- Optional port (1–65535).
- 🎲 Random IP button — generates `10.x.x.x` private IP + random port.
- 📡 Use my IP button — re-detects public IP via ipify.
- Recent rooms list (last 8) — click to autofill.

Room switching tears down all transports, resets state, and re-initializes on the new key.

### 5.5 Public Rooms directory (🌐)

Three-tab modal:
- **Browse** — 8 built-in rooms + user-created ones, with search + category filter.
- **+ Create Room** — name + description + category → auto-generated `pub-<slug>-<suffix>` key.
- **My Rooms** — manage rooms you've created; remove from public list (still joinable by key).

Built-in rooms:
| Key | Name | Category |
|---|---|---|
| `pub-lobby` | 🌍 Global Lobby | general |
| `pub-tech` | 💻 Tech Talk | tech |
| `pub-random` | 🎲 Random | random |
| `pub-gaming` | 🎮 Gaming Lounge | gaming |
| `pub-music` | 🎵 Music Room | music |
| `pub-study` | 📚 Study Hall | study |
| `pub-language` | 🗣️ Language Exchange | language |
| `pub-memes` | 😂 Memes | random |

### 5.6 Share modal (🔗)

- **QR code** — 200×200 PNG, pure-JS implementation (visual placeholder; for spec-compliant QR see §11.5).
- **Share link** — `https://your-host/#room=<encoded-ip[:port]>`.
- **Copy buttons** — link and room key.
- **⬇️ QR PNG** — download QR as PNG file.
- **📤 Native share** — `navigator.share()` on mobile, clipboard copy on desktop.

### 5.7 Deep-link auto-join

Any URL with `#room=<key>` auto-joins that room on page load (after 800 ms boot delay). Useful for embedding room links in tweets, emails, posters, QR codes.

### 5.8 Theming

Three-way theme switcher:
- 🌙 Dark — `#0a0a0f` background, purple-teal accents.
- ☀️ Light — `#f5f6fa` background, muted accents.
- 💻 Auto — follows `prefers-color-scheme` media query.

Theme updates the `<meta name="theme-color">` tag so mobile browser chrome matches.

### 5.9 Settings (all stored locally)

| Setting | Default | Effect |
|---|---|---|
| Theme | `auto` | Dark / Light / Auto |
| Sound notifications | on | Tone on new message |
| Browser notifications | off | OS push when tab hidden |
| Enter to send | on | `Enter` sends; off → `Ctrl/Cmd+Enter` |
| Show system messages | on | Toggle join/leave/rename events |
| 🌍 Global mode | **on** | WebRTC + MQTT worldwide |
| Custom WebSocket relay | off | Extra relay (in addition to global) |
| Custom relay URL | (blank) | Override default; `<room>` placeholder auto-substituted |
| Clear local data | — | Wipes all `ipchat.v1.*` keys and reloads |
| Export chat | — | Downloads JSON snapshot |

---

## 6. Keyboard Shortcuts & Commands

### 6.1 Keyboard shortcuts

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
| Open URL with `#room=<key>` | Auto-join that room on page load (deep-link) |

### 6.2 Slash commands

| Command | Action |
|---|---|
| `/help` | List available commands |
| `/me <action>` | Send an action (italicized) |
| `/nick <name>` | Change your nickname |
| `/who` | List online users |
| `/clear` | Clear your local message view |
| `/theme dark\|light\|auto` | Switch theme |
| `/topic <text>` | Set room topic (shown in header) |
| `/join <ip[:port]>` | Switch to a manual room |
| `/room` | Show current room key |
| `/global on\|off` | Toggle global mode |

---

## 7. Security & Privacy

### 7.1 What is stored locally

All under the `ipchat.v1.` namespace in localStorage:

| Key | Value |
|---|---|
| `nickname` | Current nickname string |
| `userId` | Persistent UUID |
| `settings` | JSON object of all settings |
| `messages.<room>` | Array of last 200 messages (one bucket per room) |
| `nickname-prompted` | Boolean — whether first-run nick modal was shown |
| `lastRoom` | `{ip, room, port}` of the most recently joined room |
| `recentRooms` | Array of last 8 `{ip, room, port}` entries for the Join modal |
| `publicRooms` | Array of user-created public room objects |
| `myRooms` | Array of room keys the current user has created |

**Settings → Clear local data** wipes every key matching `ipchat.v1.*` in one click.

### 7.2 What is transmitted

| Destination | What's sent | When |
|---|---|---|
| **ipify API** (`api.ipify.org`) | GET request, no body/headers | Once on init (and on 📡 Use my IP click) |
| **BroadcastChannel** | Full message payloads | Stays in browser process — never leaves the device |
| **PeerJS broker** (`0.peerjs.com`) | Signaling data (SDP/ICE) only; chat content flows P2P | While global mode is on |
| **EMQX broker** (`broker.emqx.io`) | Full message payloads in plaintext JSON | While global mode is on |
| **Custom WebSocket relay** (if configured) | Full message payloads in plaintext JSON | While relay toggle is on |

### 7.3 Trust boundaries

| Layer | Trusted? | Reason |
|---|---|---|
| `localStorage` | Local only | Stays on this device |
| `BroadcastChannel` | Local only | Same browser, different tabs |
| ipify API | Trusted for IP only | Only receives your IP string |
| PeerJS broker | Trusted for signaling only | Sees connection setup, not message content (P2P) |
| EMQX broker | **Untrusted** | Public service, plaintext, no auth |
| Custom WebSocket relay | Depends on operator | You control it if self-hosted |
| Other peers | **Untrusted** | Anyone can claim any nickname; no auth |
| Share URL / QR | Public | Anyone who scans/opens the link joins |
| Public Rooms directory | Per-browser | Custom rooms only visible to users on the same browser |

**There is no authentication, no nickname reservation, no message signing.** This is intentional — the app is for casual chat, not for sensitive communication. Sharing a room link or QR code is equivalent to sharing the room key.

### 7.4 Input sanitization

- All user-supplied text is escaped via `escapeHtml()` before being inserted into the DOM.
- Markdown renderer escapes first, then re-applies formatting tags — so `<script>` tags in messages render as inert text.
- Links get `target="_blank" rel="noopener"`.
- Nicknames are capped at 24 characters.

### 7.5 Recommendations

- **Don't share sensitive information** in any room — assume anything you send could be read by the EMQX broker operator, the PeerJS signaling server, or anyone subscribed to your MQTT topic.
- **Use random IP rooms** for sensitive chats — clicking 🎲 Random IP generates a `10.x.x.x:port` key that's effectively unguessable.
- **Turn off global mode** if you only need same-browser sync — broadcasts stay local, no external traffic.
- **Self-host a WebSocket relay** for guaranteed privacy — your relay, your rules.

---

## 8. Technical Implementation

### 8.1 Stack

- **Vanilla JS** — no framework, no build step, no dependencies. Runs from a single `.html` file (~1300 lines of JS, ~500 lines of CSS).
- **CSS custom properties** for theming via `:root[data-theme="..."]` selectors.
- **CSS Grid + Flexbox** for layout.
- **Web APIs**: `BroadcastChannel`, `WebSocket`, `RTCPeerConnection` (via PeerJS), `MQTT` (via mqtt.js), `localStorage`, `crypto.subtle`, `crypto.randomUUID`, `AudioContext`, `Notification`, `navigator.clipboard`, `navigator.share`, `canvas.toDataURL`, URL hash routing, dynamic `<script>` tag loading.
- **External CDN libraries** (loaded only when global mode is on):
  - `peerjs@1.5.4` (~30 KB) — WebRTC abstraction layer.
  - `mqtt@5.3.5` (~50 KB) — MQTT client.
- **QR code** — pure JS `drawQR()` implementation (visual placeholder, not spec-compliant; see §11.5 for upgrade path).

### 8.2 Performance choices

- **Append-only rendering** — new messages are inserted as individual DOM nodes rather than re-rendering the whole list. `renderAll()` is only called on initial load or `/clear`.
- **Cross-transport dedup** — `seenIds` Set with FIFO eviction at 2000 entries prevents double rendering.
- **localStorage writes throttled** to one per message add.
- **Reaction updates** patch the DOM in place rather than re-rendering the parent message.
- **Online strip** re-renders only on presence-tick or join/leave event, not on every message.
- **Typing indicator** is throttled to once per 1.5 seconds per sender.

### 8.3 Resilience

| Failure mode | Behavior |
|---|---|
| ipify unreachable | Falls back to `api64.ipify.org`, then to fingerprint hash |
| PeerJS broker down | WebRTC transport shows `err`; MQTT + WebSocket still work |
| Hub peer disconnects | Remaining peers race to claim hub ID; winner becomes new hub |
| MQTT broker drops | Auto-reconnect with 3-second backoff |
| WebSocket relay drops | Auto-reconnect every 3 seconds while relay toggle is on |
| `BroadcastChannel` unsupported | `S.bc = null`, falls back to single-tab mode silently |
| `localStorage` quota exceeded | Writes wrapped in try/catch, app keeps running |
| Stale peer (no heartbeat 45s) | Pruned from online map on next presence tick |
| Tab backgrounded | Sound + browser notifications fire; tab title shows unread count |
| Manual join with bad IP | Inline validation rejects; toast explains the format expected |
| Manual join with same IP:port as current | Treated as a no-op re-init; same room key, no data loss |
| Switching rooms while transports are up | All transports torn down (BroadcastChannel closed, PeerJS destroyed, MQTT ended, WebSocket closed) before the new ones connect |
| Deep-link hash invalid or empty | Falls back to default IP-based room silently |
| `navigator.share` unsupported | Falls back to clipboard copy with toast notification |
| `navigator.clipboard` blocked | Toast still appears but copy may silently fail; user can manually select+copy the link text |
| Public Rooms directory empty | Shows "No rooms found. Try creating one!" placeholder |
| My Rooms tab empty | Shows "You haven't created any rooms yet." placeholder; Remove button disabled |
| PeerJS lib fails to load | WebRTC transport stays off; MQTT and WebSocket still work |
| MQTT lib fails to load | MQTT transport stays off; WebRTC and WebSocket still work |
| WebRTC blocked by firewall | WebRTC shows `err`; MQTT picks up the slack |

### 8.4 Browser compatibility

Targets modern evergreen browsers (Chrome 90+, Firefox 90+, Safari 15+). Key APIs required:

- `BroadcastChannel` (Safari 15.4+)
- `crypto.randomUUID` (with fallback to manual UUID)
- `backdrop-filter` (Safari 9+, graceful degradation)
- CSS `env(safe-area-inset-*)` (iOS 11.2+)
- `100dvh` viewport unit (modern mobile browsers)
- `WebSocket` (universal)
- `WebRTC` (universal in modern browsers; PeerJS handles vendor prefixes)
- `Intl.DisplayNames` (for country name display; falls back to code)
- `navigator.clipboard` (Chrome 66+, Firefox 63+, Safari 13.1+)
- `navigator.share` (mobile Safari, Chrome Android; falls back to clipboard on desktop)

---

## 9. Use Cases

### 9.1 Home network

Family members on the same Wi-Fi can chat without installing anything or sharing links. Open the URL — you're in the same room automatically because you share a public IP.

### 9.2 Office LAN

Coworkers on the same corporate network get a shared ephemeral channel. No IT approval needed since there's no server to provision. Messages vanish in 24h, leaving no audit trail on disk.

### 9.3 Café / public hotspot

Anyone connected to the same café Wi-Fi can drop into the room. Useful for impromptu coordination ("whose laptop is this?") without exchanging contact info.

### 9.4 Cross-network friendships (global mode)

Two friends on different continents install ipchat. With global mode on (default), they connect via WebRTC + MQTT and chat in real time. No VPN, no server, no signup. Just share the room key.

### 9.5 Throwaway / anonymous rooms

Click 🎲 Random IP in the Join modal to generate a fresh `10.x.x.x:port` key. Share with whoever you want. After the conversation ends, join another room — the old one is abandoned.

### 9.6 Public communities

Click 🌐 Public Rooms → Browse to discover free open-access rooms. The 8 built-in rooms act as universal gathering points. Create your own under + Create Room and it'll be added to the directory (visible to anyone on the same browser; future relay-based sync could expose it globally).

### 9.7 Shareable room links & QR codes

Click 🔗 Share to generate a `#room=<key>` URL or download a QR PNG. Useful for:
- Posting a room link in a Discord/Slack channel for instant join
- Printing a QR code on a physical poster or sticker for scan-to-join at conferences, classrooms, events
- Sending via `navigator.share()` directly into WhatsApp/iMessage on mobile

### 9.8 Deep-linked landing pages

Embed `#room=<key>` in any URL — your marketing site, a tweet, a QR code on a flyer — and visitors who click it auto-join that specific room on load. No buttons to click, no setup. Great for event-specific chat rooms tied to a campaign.

### 9.9 Development / testing

Useful as a zero-config demo of BroadcastChannel + WebRTC + MQTT patterns for developers learning real-time web protocols. The single-file architecture makes it easy to inspect and modify.

### 9.10 Dorm / shared housing

Roommates on a shared router get a persistent group chat without needing WhatsApp/Discord groups. The 24h TTL keeps the chat fresh — yesterday's lunch plans don't linger forever.

### 9.11 Language exchange

Join the built-in `pub-language` room. With country flags visible next to each user, you can see at a glance who's from where. Practice languages with native speakers — the 24h TTL keeps conversations focused.

---

## 10. Limitations & Troubleshooting

### 10.1 Limitations

| Limitation | Reason | Mitigation |
|---|---|---|
| Same-IP requirement (auto mode) | Default room key derives from public IP | Use 🔑 Join modal or `/join` command to enter any custom IP+port |
| No cross-network messaging without global mode | WebRTC + MQTT must be enabled | Global mode is ON by default; can be toggled in Settings |
| No message search | Ephemeral by design | Export to JSON for archival |
| No media / file sharing | Out of scope for v1 | Could be added via relay-side storage (see §11.3) |
| No nickname ownership | No auth layer | Acceptable for casual use |
| MQTT is plaintext | Public free broker | Use custom WebSocket relay with TLS + auth for sensitive content |
| Public Rooms directory is per-browser | No backend stores custom rooms | Built-in 8 rooms are universal; for cross-browser sync, route room listings through the relay (see §11.5) |
| QR code is not spec-compliant | Pure-JS placeholder implementation | Swap `drawQR()` for `qrcode.js` library for real scannable codes (see §11.5) |
| Share link reveals room key | URL hash is visible to anyone with the link | Acceptable for casual use; for private rooms, use a less guessable key (e.g. `10.x.x.x:random-port`) |
| WebRTC may be blocked by corporate firewalls | UDP/STUN traffic is restricted | MQTT picks up the slack; or configure a custom WebSocket relay |
| No delivery confirmation | Best-effort messaging | Watch the typing indicator and replies for indirect confirmation |

### 10.2 Troubleshooting

#### "I can't see anyone in the online strip"
1. Check the **global badge** in the header — if it says `local only` or `connecting…`, transports are still coming up. Wait 5–10 seconds.
2. Open Settings → **Live transports** panel. All four transports should show `on` (or `off` for relay if not configured).
3. If WebRTC shows `err`, your network may block PeerJS. Try a different network.
4. Refresh the page — transports re-initialize on reload.

#### "Messages aren't reaching my friend on a different network"
1. Both of you must have **🌍 Global mode ON** (default).
2. Both must be in the **same room key** — verify by clicking 🔗 Share and comparing the room key.
3. Check **Live transports** — at least one of WebRTC/MQTT must be `on` for both of you.
4. If behind a corporate firewall, configure a custom WebSocket relay (§3.7).

#### "My messages are duplicated"
This was a known issue in earlier versions but should be fixed by the `seenIds` deduplication. If you still see duplicates:
1. Hard refresh the page (Ctrl+Shift+R).
2. Clear local data (Settings → 🗑️ Clear local data) and re-join.

#### "The QR code doesn't scan"
The `drawQR()` implementation is a visual placeholder, not a spec-compliant QR code. The shareable URL is the source of truth — copy it via the **Copy** button and paste into a real QR generator if you need a scannable code.

#### "Sound notifications aren't playing"
- Check Settings → **Sound notifications** is on.
- Browser may block autoplay audio until you interact with the page. Click anywhere on the page first, then have someone send a message.
- Mobile browsers (especially iOS) may require the page to be in the foreground.

#### "Browser notifications don't appear"
- Settings → **Browser notifications** must be on.
- You must grant OS-level permission when prompted.
- Some browsers require HTTPS for notifications — make sure you're serving ipchat over HTTPS, not HTTP or `file://`.

#### "I'm seeing someone with the wrong country flag"
Country detection uses `navigator.language` first, then timezone. Both can be wrong if:
- The user has set a non-native browser language (e.g. an American in Tokyo with `en-US`).
- The timezone-to-country dictionary doesn't include their city.
- The user is on a VPN that changes their apparent timezone.

This is cosmetic only — doesn't affect messaging.

#### "The app is slow on mobile"
- WebRTC connections can be battery-intensive on mobile. Turn off global mode (Settings → 🌍 Global mode) if you don't need cross-network messaging.
- The 24h message TTL can grow large in busy rooms. Run `/clear` periodically, or Settings → 🗑️ Clear local data.

#### "I can't delete someone else's message"
That's by design — only the sender can delete their own messages. You can react to any message (👍 or 👎), but not remove it.

---

## 11. Extending ipchat

The codebase is structured to make several extensions straightforward.

### 11.1 Custom relay

Point at your own WebSocket server via Settings → Custom relay URL. Use the `<room>` placeholder:

```
wss://your-relay.example.com/chat/<room>
```

The relay only needs to broadcast incoming JSON to all connected clients — no message inspection required.

### 11.2 End-to-end encryption

The message bus is agnostic to payload encoding. A future version could:

1. Exchange ECDH public keys on `hello`.
2. Derive a shared room key per peer pair.
3. Encrypt `msg.text` before `remoteSend()`.
4. Decrypt on receive.

The renderer would need minimal changes — just decrypt before `renderMarkdown()`.

### 11.3 File / image sharing

The wire protocol already supports arbitrary `kind` fields. Adding `kind: 'image'` with a base64 payload (capped at relay message size limits) would require:
- Composer: file input + drag-drop handler.
- Renderer: `<img>` branch in `renderMessage()`.
- Persistence: optional — large media could be excluded from localStorage.

### 11.4 Persistent rooms

Replacing `roomFromIp(ip)` with a hash of a user-provided room name would enable invite-based rooms independent of IP, while keeping the rest of the stack intact.

### 11.5 Cross-browser public directory & real QR codes

The Public Rooms directory currently lives only in localStorage, so two users on different browsers see different custom rooms. To make the directory truly global, route room listings through the existing MQTT transport:

1. Add `type: 'room-list-request'` to the message protocol.
2. Peers respond with their local `publicRooms` array.
3. Merge into a deduped global directory; cache for 5 minutes.
4. Optionally, persist popular rooms (top-N by recent activity) in localStorage for offline access.

For the QR code, the current `drawQR()` produces a visual placeholder. To make it spec-compliant and scannable by all phone cameras:

1. Vendor in a tiny QR library (~10 KB) such as `qrcode-generator` or `qrcode.js`.
2. Replace the body of `drawQR()` with a call to the library's render function, targeting the same `<canvas>` element.
3. The rest of the Share modal needs no changes — the link, copy button, native share, and PNG download all work as-is.

### 11.6 Custom country detection

If you want more accurate country detection (e.g. via a paid IP geolocation API), replace the body of `detectCountry()` with a fetch to your API of choice:

```javascript
async function detectCountry() {
  const r = await fetch('https://your-geo-api.com/?key=YOUR_KEY');
  const j = await r.json();
  return j.country_code;
}
```

The rest of the app already handles async country detection — it just needs to be awaited in `init()` and propagated to all peers via the `country` field on every outgoing message.

### 11.7 Presence-based features

The presence system already broadcasts `country` and `locale` for each user. Future features could include:
- **Country-based room filtering** in the Public Rooms directory (show only rooms with peers from your region).
- **Language matching** — pair users with similar `locale` for language exchange.
- **Timezone-aware "quiet hours"** — mute notifications during the recipient's nighttime.

---

## 12. Appendix

### 12.1 Quick command reference card

```
HELP        /help                    Show available commands
IDENTITY    /nick <name>             Change your nickname
            /who                     List online users
MESSAGING   /me <action>             Send an action (italicized)
            /topic <text>            Set room topic
NAVIGATION  /join <ip[:port]>        Switch to a manual room
            /room                    Show current room key
APPEARANCE  /theme dark|light|auto   Switch theme
            /clear                   Clear your local view
TRANSPORTS  /global on|off           Toggle worldwide mode
```

### 12.2 Default settings

| Setting | Default |
|---|---|
| Theme | `auto` |
| Sound notifications | `on` |
| Browser notifications | `off` |
| Enter to send | `on` |
| Show system messages | `on` |
| 🌍 Global mode | **`on`** (new default — WebRTC + MQTT) |
| Custom WebSocket relay | `off` |
| Custom relay URL | (blank — uses `wss://socketsbay.com/wss2/<room>` if relay is enabled) |

### 12.3 Built-in room keys

| Key | Display name |
|---|---|
| `pub-lobby` | 🌍 Global Lobby |
| `pub-tech` | 💻 Tech Talk |
| `pub-random` | 🎲 Random |
| `pub-gaming` | 🎮 Gaming Lounge |
| `pub-music` | 🎵 Music Room |
| `pub-study` | 📚 Study Hall |
| `pub-language` | 🗣️ Language Exchange |
| `pub-memes` | 😂 Memes |

### 12.4 Glossary

| Term | Meaning |
|---|---|
| **Room key** | The normalized string (e.g. `ipchat-203-0-113-50`) that identifies a chat room. All peers with the same key can see each other. |
| **Transport** | A communication channel: BroadcastChannel, WebRTC, MQTT, or WebSocket. ipchat uses all four in parallel. |
| **Hub** | In the WebRTC mesh, the peer that owns the hub ID `iphub-<room>` and re-broadcasts messages to all connected clients. |
| **Deep-link** | A URL containing `#room=<key>` that auto-joins the specified room on page load. |
| **Global mode** | Toggle for WebRTC + MQTT transports. ON = worldwide messaging. OFF = local-browser-only. |
| **TTL** | Time-to-live. Messages have a 24h TTL and are purged on every read/write. |
| **Heartbeat** | Periodic `presence` message sent every 15 seconds to keep the online list fresh. |
| **Soft-delete** | A message marked as deleted but kept in localStorage with a "🗑️ message deleted" placeholder. |

### 12.5 Default relay endpoints

| Service | URL | Purpose |
|---|---|---|
| **ipify** | `https://api.ipify.org?format=json` | Public IP detection |
| **ipify v6** | `https://api64.ipify.org?format=json` | IPv4 + IPv6 fallback |
| **PeerJS broker** | `0.peerjs.com` (handled by `peerjs.min.js`) | WebRTC signaling |
| **EMQX broker** | `wss://broker.emqx.io:8084/mqtt` | MQTT pub/sub |
| **Default WebSocket relay** | `wss://socketsbay.com/wss2/<room>` | Optional cross-device relay |

### 12.6 Final notes

ipchat is a single-file demo of how much real-time web functionality can be packed into a browser with zero backend. Every feature — global messaging, public rooms, QR sharing, deep-linking — works without any server code you control. The trade-offs (best-effort delivery, no auth, ephemeral storage) are deliberate choices that keep the app simple, free, and instantly usable.

For sensitive or production use cases, self-host a WebSocket relay and consider adding end-to-end encryption (see §11.2). For everything else, the default configuration works out of the box — open the file, set a nickname, and start chatting with the world.
