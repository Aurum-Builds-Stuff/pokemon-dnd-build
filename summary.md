# Pokémon D&D Campaign — Project Summary

## Project Overview
Building a web app companion for a 3-player Pokémon-themed D&D campaign ("The Cataclysm Campaign"). DM runs a control panel; each of 3 players has their own device showing a read-only/live-synced trainer view. Physical dice are rolled on the DM's phone (not built into the app) and results entered manually.

## Tech Stack & Hosting
- **Frontend**: single static `index.html` file (vanilla JS, ES modules), no build tools — hosted free on **GitHub Pages**.
- **Backend/sync**: **Firebase Realtime Database** (free Spark tier) + **Anonymous Auth**. DM writes state, all player screens sync live via `onValue` listeners.
- **Firebase Storage was rejected** — Google now requires the paid Blaze plan even for free-tier usage (changed Feb 2026). Instead, story-scene and sprite images are hosted directly in the GitHub repo as static files.
- Database rules currently set open (`.read`/`.write`: true) for development — **must be locked down before real campaign play**.
- Firebase config is already in the code (project: `pokemon-dnd-build`).

## Repo Structure
```
index.html          (the whole app)
scenes/              (47 story illustration PNGs, DM-broadcastable)
sprites/             (31 uploaded NPC/trainer PNGs, 80x80, transparent PNG)
```

## Visual Style
Retro pixel-game aesthetic. Fonts: "Press Start 2P" (headers) + "VT323" (body), via Google Fonts CDN. Palette: navy `#14142b` bg, panel `#1f1f3d`/`#2a2a52`, red `#e63950`/`#a5253a`, gold `#ffd23f`, teal `#4ecdc4`, text `#f4f1de`. Chunky pixel borders, no border-radius, boot-screen title card.

## Story: "The Cataclysm Campaign"
Full story doc already uploaded and processed. Key structure:
- **8 Gyms**: Brock(Rock)→Misty(Water)→Clemont(Electric)→Flannery(Fire)→Cilan(Grass)→Olympia(Psychic)→Allister(Ghost)→Iris(Dragon). Rosters range 3→18 Pokémon, multiples of 3.
- **Battle modes**: "rotation" (P1→P2→P3, defeated player sits out, next continues — normal gyms) vs "simultaneous" (all 3 vs 1 — bosses, Gladion, Iris, Elite Three, Champions).
- **HP system**: Full→½→¼→⅛→Defeated (one stage per successful hit, no fraction math).
- **Status effects** (confuse/poison/burn): all identical — skip next turn.
- **Team Cataclysm**: 5-beat villain arc (Cerulean Lab kidnapping → Blue Orb heist Lumiose → Red Orb from Flannery's betrayal → Green Orb reveal Route 6 → Tempest Island finale). Players are *meant* to fail these story battles (no restart) — DM will flag which battles are scripted losses.
- **Route 3**: co-op boss "Backpacker Talon" (Machoke/Haunter/Gabite).
- **Route 6**: Gladion boss fight (Silvally/Lucario/Zoroark).
- **Mystery Egg** (Route 7): 3-way player battle, winner's egg hatches immediately — **user wants to change reward from Larvitar to Ash-Greninja** (PokeAPI slug: `greninja-ash`).
- **Mega Stone Shrine** (Route 8): 3 separate guardian challenges (Kira/Ronan/Mei), each player gets own generic Mega Stone (not shared/passed). Only 1 Mega Evolution per battle per team.
- **Final Arc**: Weather Trio (Groudon/Kyogre/Rayquaza) awakened at Tempest Island; each player solo-fights one Legendary (with themed status effect: burn/confuse/paralysis), catch option opens at ⅛ HP.
- **League**: Elite Three (Lance/Flint/Glacia, rotation) → 3 Champions (Cynthia/Steven/Diantha, simultaneous 3v1) → **Final Championship**: round-robin P1vP2/P1vP3/P2vP3, most wins = True Champion; 1-1-1 tie → free-for-all.
- Routes 1–4 rosters/encounter tables were drafted by Claude (low-tier, escalating difficulty) since original doc left them unspecified.

## Game Rules (finalized)
- **Starters**: roll d20 (external dice), lowest picks first; each player picks a **type** (Fire/Grass/Water, no repeats) then a **starter** from that type across all 9 gens.
- **Catching**: DM manually picks route → player → specific Pokémon from that route's encounter table (no RNG/formula) → banner shows on player's screen → DM taps **Catch!** / **Escaped** (ball consumed either way) / **Player Ran Away** (no ball consumed, added per user's last request).
- **Party cap**: 6 (overflow → Box).
- **Healing**: automatic on city arrival.
- **Pokéballs**: DM-granted only (gym/NPC/chest), tracked per player with a "+1 ball" button in DM roster.
- **Battle loss**: rotation battles don't "restart" (just cycle players); full-wipe battles reset via a DM "heal all & restart" button — except scripted story losses (DM will flag these, not yet built).
- **Trainers choose which Pokémon to send into battle** (not automatic).
- **Badges**: small tracker/panel per player (8 dots, fills as gyms clear) — implemented.
- **Save/resume**: continuous auto-sync (Firebase) + same code always resumes; manual checkpoint save was discussed conceptually, not yet built.
- **Delete campaign**: DM button added, with confirm dialog, permanently wipes the Firebase session node.

## Image Generation Style (user's own scene generation, not Claude's)
User is generating their own 47 scene PNGs externally using this style (from their own prompt book, uploaded as PDF): polished Pokémon-inspired fantasy adventure, anime-influenced characters, cinematic composition, rich color, no text/UI/watermarks. (Note: this superseded an earlier retro-pixel-art suggestion Claude gave — user's actual scenes use the cinematic anime style instead.)

## App Build Status (Phases 1–3 done, in `index.html`)

**Phase 1 — Core sync skeleton** ✅
- Role select (DM / Player) screen.
- DM: create new session (random 6-char code) or resume via code.
- Player: join via code, pick open slot (P1/P2/P3), enter name.
- Live sync via Firebase RTDB; local identity persisted via `localStorage` (auto-reconnects on refresh).
- "Leave / switch role" link on both DM and Player screens (clears local identity, does NOT delete server data).
- "Delete This Campaign" button (DM only, confirm dialog, permanently removes Firebase session).

**Phase 2 — Starter draft + Scene broadcaster** ✅
- DM enters each player's die roll manually → computes draft order (lowest first, catches ties) → sequential turn-based type+starter picking live on each player's own screen (already-taken types disabled).
- `STARTERS` object embedded in JS (9 starters × 3 types).
- DM Scene picker: dropdown of all 47 scenes (numbered, titled) → "Show Scene"/"Clear Scene" → full-bleed overlay (`<img>` fixed position) appears on all player screens, `<img src="scenes/{filename}.png">`.
- **Known issue just found, not yet fixed**: several scene filenames in the embedded `SCENES` array don't match what the user actually uploaded to GitHub (user's files use shorter names). Mismatches identified from user's screenshot:
  - `08-cerulean-lab` (not `-team-cataclysm`)
  - `10-route-3` (not `-boss-battle`)
  - `12-lumiose-museum` (not `-blue-orb-heist`)
  - `14-route-4` (not `-cave-approach`)
  - `15-echoing-caverns` (not `-ancient-carvings`)
  - `21-route-6` (not `-gladion`)
  - `23-route-7` (not `-ancient-monument`)
  - `28-route-8` (not `-hidden-shrine`)
  - `30-iriss-dragon-gym` (note: double-s, not `iris-dragon-gym`)
  - `31-the-beam` (not `-cataclysm-begins`)
  - `40-lance`, `41-flint`, `42-glacia`, `43-cynthia`, `44-steven`, `45-diantha` (all shortened, no `-elite-three`/`-champion-battle` suffix)
  - `46-final-championship` (not `-player-rivalry`)
  - **Scenes 24, 25, 35 are being skipped by the user** (not being generated) — need to remove from the SCENES dropdown array entirely so DM doesn't try to show missing files.
  - **This needs to be fixed next**: update the `SCENES` array in `index.html` (and ideally `campaign-data.json`) to match exact uploaded filenames, and remove entries 24/25/35.

**Phase 3 — Wild encounters** ✅ (mostly)
- `ROUTES` array embedded in JS with encounter tables for routes 1,2,4,5,6,7,8 (route 3 has no wild table, boss-only).
- DM: select route → select player → click a Pokémon from the flattened encounter table (grouped by tier/type) → writes `session.currentEncounter` → player sees "A wild {X} appeared!" banner → DM resolves with Catch!/Escaped/Player Ran Away buttons.
- Catch adds to Party (or Box if Party full) and decrements Pokéballs.
- Player screen shows live Party list, Box list, Pokéball count.
- **Bug just fixed**: a `const SCENES = [` declaration was accidentally deleted during an earlier edit, causing `Uncaught SyntaxError: Unexpected token ':'` — fixed and verified with `node --check`.
- **User reports encounter table not showing all entries** (e.g. expected 18 rows for a route, not all appearing) — **this was being actively investigated when the conversation ended, not yet resolved**. Was checking the embedded `ROUTES` JS object for route-5/6/7 for duplicate keys or truncation; initial grep showed the data itself looks complete (18 type keys + 1 ecosystem key per route). Root cause not yet found — could be a rendering/display issue (CSS overflow, cut off visually) rather than a data issue. **Next step: investigate DOM rendering of `renderEncounterTable()` function and check for any CSS clipping on `.cartridge` or `#dm-encounter-table`.**

## Outstanding / Requested Work (not yet built)

1. **Fix scene filename mismatches** (see list above) — straightforward data fix.
2. **Remove scenes 24, 25, 35 from dropdown** (user skipped generating these).
3. **Investigate/fix encounter table not showing all entries** — in progress, root cause unknown.
4. **Add a Map viewing option** — user wants a digital map view (separate from story scenes). Suggested approach (not yet built): either treat it as a persistent/pinned overlay separate from `currentScene` (e.g. new `session.mapVisible` boolean + `map.png` file), toggled via DM "Show Map"/"Hide Map" buttons, distinct from the one-shot scene broadcaster.
5. **Battle engine — NOT YET DESIGNED IN DETAIL.** User explicitly asked to co-design this "side by side" with Claude asking clarifying questions as they go, covering:
   - Which player battles when, turn order logic for both rotation-mode and simultaneous-mode fights.
   - How enemy HP scales up when all 3 players fight together (simultaneous mode) vs solo.
   - How trainers select which Pokémon to send in each round.
   - Damage application UI (HP stage tracker, tap-to-reduce).
   - This is the next major conversation topic and needs a fresh design discussion — **should be tackled interactively, asking the user questions rather than building it unprompted**, per their explicit request.
5. **Ash-Greninja swap for Mystery Egg reward** — easy: no custom sprite needed since PokeAPI has this as a form (slug: `greninja-ash`), auto-fetchable once sprite system (#6) is built. Just update campaign-data.json / story text reference from Larvitar → Ash-Greninja.
6. **Pokémon sprites in party/box lists — NOT YET BUILT.** Plan discussed but not implemented: fetch sprites live from PokeAPI (`https://pokeapi.co/api/v2/pokemon/{slug}`, use `sprites.front_default`), with a slugify function handling basic names plus a small override dictionary for special forms (Mega X → `x-mega`, Alolan X → `x-alola`, Galarian X → `x-galar`, Hisuian X → `x-hisui`, Ash-Greninja → `greninja-ash`). Should cache results client-side (e.g. a JS `Map`) to avoid refetching. Needs to render as `<img>` elements in the existing party/box `<li>` rendering (currently plain text only).
7. **NPC/trainer sprites integration** — 31 sprites already uploaded to `sprites/` folder by user, but **not yet wired into the app** (no UI currently displays them — gym leader battles, NPC rosters, Team Cataclysm scenes etc. aren't built yet). Full sprite list was given to user earlier (gym leaders, Elite Three, Champions, Gladion, professors, Team Cataclysm/Vesper, generic grunt, route NPCs, shrine guardians, 3 player trainer avatars `trainer-p1/2/3.png`).
8. **Security rules lockdown** — currently wide open, flagged as needed before actual game night but deferred.
9. **Manual save checkpoints** — conceptually agreed (separate from continuous auto-sync) but not implemented; only continuous sync + code-based resume currently exist.
10. **Full battle system per Section 4 of story doc** — HP notch tracking per Pokémon, gym rotation UI, simultaneous boss battle UI, damage application — none of this UI exists yet, only the underlying rules are documented.

## Important Context for Continuing
- The user is non-technical-ish but hands-on: comfortable clicking through Firebase console, uploading files to GitHub directly via the web UI, and pasting error messages/screenshots — not writing code themselves. Give step-by-step console instructions when infra changes are needed.
- The user works in parallel (making scenes/sprites) while Claude builds — good to keep giving them parallel-track suggestions.
- Firebase Storage is off the table due to billing requirement — always use GitHub-hosted static files or Realtime-Database-stored text/JSON for any new asset needs.
- All game logic/rules decisions have been very iterative — the user refines rules turn-by-turn (e.g. added "Player Ran Away" option, changed mega stone mechanic from shared to per-player). Expect continued rule refinement mid-build.
- The full canonical rules/story reference is `campaign-data.json` (in outputs) — should be kept in sync with `index.html`'s embedded JS data (`STARTERS`, `SCENES`, `ROUTES`) whenever either changes, though currently there's drift (e.g. scene filenames) that needs reconciling.
- Battle engine design should be done conversationally with questions, not built unilaterally, per explicit user instruction.
