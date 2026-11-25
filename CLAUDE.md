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
- Fly-in animation from the player's chip. The last-played card is outlined in the player's color with a name tag. Per-seat colors.
- Board with player chips around it on top, then the turn label and Pass, your hand, the points line, the Recent line and the invite box.
- Bell at 2 cards, the Sure/Not-sure call banner and chime, and a big PASS stamp across the board (small one for other players).
- Sticky slim header with 🔊 (mutes every game sound on that phone). Slim emoji line fixed at the bottom: one tap sends a reaction; 💬 opens the text box and history at the top of the screen (unread count on it). Chat bubbles and messages show at the top.
- Round results: scores table plus each player's leftover cards.
- Leave table, Start my own table, New game (clear scores), Change players, AI players (Aaji, Kaka…).
- Respects `prefers-reduced-motion`. Mobile-first (≈390 px wide).

## Roadmap

### Phase A — game feel (done, live; adjusted by Phase A.1 below)

1. **Pace and hints**
   - Host sets the table defaults in the lobby: **Hints on/off**, **AI think time** (1–5 s, default 2 s) and **Turn timer** (off/15/20/30/45/60 s, default 20 s).
   - Each player can override the hints setting for themselves (stored locally).
   - Hints off: no glow on playable cards. Tapping an invalid card flashes a red ✗ on it and plays an error tone. The card is not played.
   - **Manual Pass button.** No more auto-pass for humans. Pressing Pass while holding a playable card gives a red ✗ and error tone ("must play if you can").
   - Timer runs out: if the player has no playable card, they pass; if they have one, the AI plays it for them. Show a countdown (~~on the turn banner~~ see A.1) and tick during the last 5 s, heard only by the player on turn.
2. **PASS stamp over the passing player's icon**, in addition to the board stamp.
3. ~~**Turn banner at the top:**~~ **Removed in A.1**, replaced by the player chips. Was: red "<name>'s turn" with the countdown → green "<name> played 8♥" for ~1.5 s → red for the next player.
4. **End-of-round show:** fanfare and a winner banner with confetti, then players revealed one at a time in ascending points, with their leftover cards.

### Phase A.1 — tweaks from the first family test (done)

1. **The top turn banner is removed.** Mini player chips sit around the edge of the card table (felt) instead, split across the top and bottom edges, up to 8 players.
   - Chip: name (shortened if needed), cards left, player color.
   - On that player's turn the chip border is red, with the turn timer as a ring shrinking around it.
   - Green for ~1.5 s after they play, with the card they played shown on or near the chip. Amber plus the PASS stamp for a pass.
   - You are always at the bottom right on your own phone; turns run clockwise (bottom row leftwards, top row rightwards). Bottom row holds half the players, rounded down.
   - Player tiles are shown in the lobby only. During a round a one-line "Points:" list replaces them.
2. **"It's your turn" must be unmistakable for the current player:** felt edge glows, hand area highlighted with a "Your turn" label and countdown, soft chime (respects the 🔊 mute button), short vibration where supported. Not distracting for others.
3. **Host table setting "Players can change hints"** (default: no). When off, the 💡 button is hidden and everyone uses the table's hint setting. Changing it applies immediately: the host can change the two hint settings mid-round too (Hint settings button). Other settings only between rounds.
4. **AI going out:** when an AI's turn would play its last card, it plays after ~1 s instead of the think time.
5. Everything else stays as is. The host can change table settings in the lobby and on the end-of-round screen without losing scores (added in Phase A).

### Phase A.2 — second round of family feedback (done)

1. Bigger pile cards: strip height and rank labels grow with the free screen height, and the highest card shows its rank and suit large.
2. Chat dock removed; chat opens from 💬 in the header.
3. The name tag on the last-played card fades after 1.5 s (the colored outline stays).
4. Turn timer default is 30 s. The countdown and ring only show for the last 15 s.
5. Gentle tone when you play a card, "wrong answer" tone when you pass, heard only on your own phone.
6. One 🔊/🔇 button replaces the Bell button and silences all game sounds (vibration is not affected).
7. **Nothing may ever cover the Sure / Not sure buttons.** They sit in the page flow above the hand; messages go to the top of the screen.

### Phase A.3 — third round of family feedback (done)

1. **Moves settle before the next turn:** card lands, name tag goes away, then the red indicator, chime, unlocked cards and timer move to the next player (1.6 s after a play, 1.2 s after a pass). The host sets `turnAt` that far ahead (`SETTLE`); each phone gates on its own animation (`live()`). AI think time counts from there.
2. **Finale:** the last card flies slowly (2.4 s) via the middle of the board, then stays highlighted with a "Final card!" caption before the winner show (4.8 s in all, 2.8 s with reduced motion).
3. **Emoji line** at the bottom, one tap to send; 💬 opens the text box, ready to type, and closes after sending.
4. Center cards grow when your hand needs only one row.

### Phase A.4 — flying reactions (done)

- A reaction flies from the sender's chip (lobby: their tile; spectators: bottom center) to the middle of the board with a trail, the sender's name in their color, and a small badge on their chip for 4.5 s. Reduced motion: it just appears at the chip.
- State: `rxs`, the last 8 reactions `{id, k, w, e}`. Each phone shows every id once. **Never compare host timestamps with a phone's own clock to decide whether to show something** (use ids, or `now()` on both sides). `rx` is still written for older pages.
- Anything drawn over the board goes in `#fx`; inside the felt it can end up under the cards (they have z-index).

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
