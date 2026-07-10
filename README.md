# Nite — GitHub Pages build

This is a static, backend-free build of Nite Party. It's identical to the full
app except for one thing: instead of a self-hosted signalling server, it uses
PeerJS's free public cloud broker (`0.peerjs.com`) for WebRTC connection
setup — which is what makes it work with zero backend on GitHub Pages.

## Deploy

1. Create a GitHub repo (or use an existing one) and push the contents of this
   folder to it — `index.html`, `games-data.js`, `favicon.svg`, `404.html`.
2. In the repo, go to **Settings → Pages**, set **Source** to the branch/folder
   containing these files (e.g. `main` / `/root`), and save.
3. GitHub gives you a URL like `https://<username>.github.io/<repo>/` —
   that's your app. Share it directly, or share a room link once you're in a
   room (the in-app "copy link" button already uses the page's own URL).

No build step, no `npm install`, no server to run — just static files.

## How it works / limits

- **Host** creates a room and streams video/screen/audio over WebRTC.
  **Viewers** open the same URL (or a room link) and join.
- Signalling (the initial "who's who" handshake) goes through PeerJS's public
  cloud broker. The actual video/audio/game data always flows directly
  peer-to-peer (or via a TURN relay when needed) — never through PeerJS's
  servers.
- Because there's no backend, cross-network connections rely on the public
  STUN/TURN servers configured in the app (Cloudflare/Google STUN, OpenRelay
  TURN). These are free public services with no uptime guarantee. If someone
  can't connect from a very restrictive network, that's the likely reason —
  there's no self-hosted fallback in this build.
- Everything else (games, sync, scoreboard, YouTube playback, etc.) works
  exactly like the full app, since none of it depends on a server.

## Reliability hardening built in

Since a static build can't fall back on a server for anything, the app
self-heals as much as possible on the client alone:
- **Signalling reconnect** — if the connection to PeerJS's broker silently
  drops (phone locked/backgrounded, laptop sleep, brief WiFi loss), the app
  automatically reconnects with backoff, and also re-checks as soon as the
  tab regains focus or the browser comes back online — the two moments a
  dropped signalling socket is most likely to go unnoticed.
- **ICE grace period + restart** — a momentary "disconnected" WebRTC state
  (common on brief network hiccups) is no longer treated as a dead
  connection. The app gives it a few seconds to self-heal, then tries a
  lightweight ICE restart before falling back to a full rejoin — so short
  drops recover in place instead of kicking people out of the stream.
- **Data-channel reconnect** (pre-existing) — if a viewer's connection to the
  host drops outright, the app retries with exponential backoff (up to 5
  attempts) before showing a disconnected screen.

What's still out of the app's control: the free public TURN relay's uptime
and bandwidth, since there's no paid/dedicated fallback wired in for a
zero-setup static build.

## Difference from the Replit/self-hosted build

The version of `index.html` in the project root (outside this folder) is
configured to talk to a self-hosted PeerJS signalling server (`server.js`,
served at `/peerjs`) and reads a Cloudflare tunnel URL from `/api/public-url`.
This build removes both of those server dependencies so it can run as plain
static files.
