# Satti Lavani (Badam 7) — family card game

Multiplayer web game so Nikhil can play Satti Lavani (सत्ती लावणी, a.k.a. Badam 7 / Sevens) with family and relatives over WhatsApp links. No accounts for players.

- Live site: https://ndeo-git.github.io/badam7/ (GitHub Pages, `main` branch, repo root)
- Everything is in one file: `index.html` (HTML + CSS + JS, no build step). Keep it that way unless asked.
- Deploy = commit and push to `main`. Pages updates in ~1–2 minutes. The link must never change.

## Game rules (this family's house rules)

- 3–8 players, whole deck dealt round-robin.
- Holder of 7♥ opens with 7♥. Turns go in seat order.
- Each suit is built from its 7: up to K and down to A, one card at a time. Any 7 can start its suit (after 7♥ is down).
- If you can play, you must. If you cannot, you pass.
- First player out wins the round. Everyone else scores the cards left in hand: A=1, 2–10 face value, J=11, Q=12, K=13. Lowest running total leads.
- At 2 cards and again at 1 card, a player calls "Sure" or "Not sure".

## Architecture

- Libraries from CDN only: Firebase 10.14.1 compat (`firebase-app-compat.js`, `firebase-database-compat.js` from gstatic). Google Fonts: Yatra One + Mukta.
- Firebase Realtime Database, project `badam7-90b56`. The config in `index.html` is public by design. Rules allow read/write under `t/$code` only.
- Data per table at `t/<code>/`:
  - `state`: the whole game state as a **JSON string** (stored as a string on purpose, so Firebase doesn't mangle arrays or nulls).
  - `acts`: a push queue of player actions `{k: pid, t, d, nick}`. Only the host consumes and deletes them.
  - `online/<pid>`: presence, with `onDisconnect().remove()`.
  - `host`: `{k: pid, at}`, the host lease.
- **Host-authoritative.** One browser is the host. It runs the engine (`apply`, `play`, `pass`, `deal`, bots/AI) and writes `state`. Everyone else only renders `state` and pushes actions.
- **Host election:** every client runs a transaction on `host` every 5 s. You keep the lease by refreshing `at`, and you may claim it if it is empty or older than 20 s and you are a seated, online human (or the table creator). `hostOk` guards against a stale host writing after it wakes up from the background.
- Player identity is `pid` in localStorage. Names are free text (`nick`).
- Disconnected human: the AI plays for them after 8 s, and after 1.4 s once they've been gone more than 30 s.
- Tables: `?t=<code>`. "Start a new table" or "Start my own table" makes a new code (new link). Tables untouched for 12 h count as closed.
- Hands are hidden only in the UI (honor system). Known trade-off.

## Current features (keep working)

- Solitaire-style piles: 4 suit columns, A at the top down to K, each card showing its top strip, the highest card fully visible, and the 7s aligned.
- Fly-in animation from the player's tile. The last-played card is outlined in the player's color with a name tag. Per-seat colors.
- Board on top, then the turn status, your hand, player tiles, the Recent line and the invite box.
- Bell at 2 cards, the Sure/Not-sure call banner and chime, and a big PASS stamp across the board (small one for other players).
- Chat and emoji dock fixed at the bottom, chat bubbles at the top, 💬 history panel with unread count. (Chat dock space is a known compromise; leave it for now.)
- Round results: scores table plus each player's leftover cards.
- Leave table, Start my own table, New game (clear scores), Change players, AI players (Aaji, Kaka…).
- Respects `prefers-reduced-motion`. Mobile-first (≈390 px wide).

## Roadmap

### Phase A — game feel (next)

1. **Pace and hints**
   - Host sets the table defaults in the lobby: **Hints on/off**, **AI think time** (1–5 s, default 2 s) and **Turn timer** (off/15/20/30/45/60 s, default 20 s).
   - Each player can override the hints setting for themselves (stored locally).
   - Hints off: no glow on playable cards. Tapping an invalid card flashes a red ✗ on it and plays an error tone. The card is not played.
   - **Manual Pass button.** No more auto-pass for humans. Pressing Pass while holding a playable card gives a red ✗ and error tone ("must play if you can").
   - Timer runs out: if the player has no playable card, they pass; if they have one, the AI plays it for them. Show a countdown on the turn banner and tick during the last 5 s.
2. **PASS stamp over the passing player's icon**, in addition to the board stamp.
3. **Turn banner at the top:** red "<name>'s turn" with the countdown → green "<name> played 8♥" for ~1.5 s → red for the next player.
4. **End-of-round show:** fanfare and a winner banner with confetti, then players revealed one at a time in ascending points, with their leftover cards.

### Phase B — cards and players

5. Player icon: last card played, cards left and a proportional mini fan.
6. Own hand as a traditional fan, grouped by suit. First tap lifts a card, a second tap on it plays it.

### Phase C — landscape table

7. Wide screens (phone sideways, iPad, laptop) get a poker-style table with seats around the edge, built from Phase B components. Upright phones keep the current layout. Web pages can't lock orientation.

### Phase D — music

8. Optional background music, off by default, with a mute toggle. Only royalty-free audio (e.g. a Pixabay track Nikhil adds to the repo) or synthesized via Web Audio. Never copyrighted songs.

## Working rules

- Escape all player-supplied text (names, chat) before putting it into HTML.
- Keep the old state readable: add new state fields with defaults, because tables in progress may hold the older shape.
- **Test locally before pushing:** run `python3 -m http.server` and open several browser tabs. For multi-player tests without real Firebase, a small fake `firebase` shim backed by localStorage + `storage` events works well, with sessionStorage per tab for `pid`.
- After pushing, ask Nikhil to test a real game with family before starting the next phase.
