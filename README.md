# Blackout

**Pick a light. Share it and it dies.**

A real-time party game for 2–8 players, built to run on everyone's own phone
with nothing to install and no account to create.

Live page: https://claude.ai/artifact/GJamhwvMJpd7suemJLSgJc

> This branch is standalone. It shares no history with the rest of this
> repository — the game is unrelated to the PharmaSearch application.

## Rules

Five bulbs sit on the board, worth **1 to 5 points**. Every round, all players
secretly choose one.

| Situation | Outcome |
| --- | --- |
| You are alone on a bulb | You bank its points |
| Two or more share a bulb | It **blacks out permanently** — nobody scores |
| A bulb blacks out | Every surviving bulb gains **+1**, each time |

Each player also gets **one Surge per game**:

- Surge into a collision → you take the bulb **and it survives**.
- Surge a bulb you had to yourself → you score **nothing**.
- Two players surge the same bulb → it blacks out anyway, both surges spent.

The game ends when one bulb is left standing, or after nine rounds. Most points
wins; equal scores share the win.

The board destroying itself is the point. It opens wide and ends with everyone
crowded around two bulbs nobody can afford to share.

## Two ways to play

**Rooms.** One player starts a room and shares the four-character code or the
invite link. Everyone plays on their own phone, in sync.

**Pass & Play.** Everyone uses one device. The screen covers between turns so
nobody sees anyone else's choice.

The page detects which is available and routes players automatically.

## Known limitation

Rooms run on the Claude artifact `room` capability, which only admits
**signed-in** viewers — members of the artifact owner's organization and
invited guests. Someone opening a public link while signed out cannot connect,
and view-only members cannot send moves.

On joining a room the page emits a probe on a topic nobody listens to. If the
platform rejects it with `not_permitted`, that viewer is told they can only
watch, and is left out of both the player roster and host election — rather
than sitting in the lobby looking like a player and silently timing out every
round.

Pass & Play exists to cover exactly that case and works for everyone.

## How the sync works

Each phone holds a slice of the game; one is elected host and broadcasts the
authoritative board through its presence object. Host election survives the
host leaving mid-game — the next player adopts the cached state and republishes.

Moves use **commit–reveal** so no one can see a choice before locking in their
own:

1. On lock-in, your phone emits `sha256(round | bulb | surge | nonce)`.
2. Once every player has committed, the host calls a reveal.
3. Each phone emits its actual pick plus the nonce.
4. The host rehashes and discards anything that does not match its commitment.

Including the round number in the hashed payload stops a commitment being
replayed into a later round.

## Working on it

`blackout.html` is the whole game — no build step and no dependencies beyond
three Google Fonts. The file is written for the Claude artifact publish
pipeline, which wraps it in a document skeleton, so it deliberately has no
`<!doctype>`, `<html>`, `<head>` or `<body>` tags of its own.

To open it directly in a browser, wrap it first:

```sh
{ printf '%s' '<!doctype html><meta charset=utf8><meta name=viewport content="width=device-width,initial-scale=1">'
  cat blackout.html
} > /tmp/blackout-local.html
```

Room play will not work from a local file — `window.claude` only exists inside
the artifact viewer — so a local copy always falls back to Pass & Play.
