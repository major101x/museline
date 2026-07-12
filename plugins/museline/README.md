# 🎭 museline

A rotating, personality-driven **statusline for Claude Code**. Instead of a plain
status row, museline cycles through short lines with real character:

```
🐋 Random fact: a bowhead whale can live over 200 years.
🤔 What is the meaning of it all? (still buffering an answer...)
🎤 Did you know? Kanye once spent a reported $10k+ on a single music video sofa.
🌀 Reticulating splines...
```

Four built-in **personality packs** (`philosophy`, `trivia`, `popculture`,
`witty`) get mixed and rotated live while Claude works. Optionally, plug in **your
own LLM API key** and museline will generate fresh on-theme lines in the background.

> **Why a statusline and not the spinner word?** Claude Code's spinner verbs
> ("Enchanting...") are hard-coded and have no plugin hook. The statusline is the
> supported, update-proof surface: it re-runs on a timer (even mid-work), so museline
> rotates right alongside the spinner. Your line is the star; the built-in verb
> just keeps it company.

## Install

```
/plugin marketplace add major/museline        # or your fork's repo
/plugin install museline@museline-marketplace
```

Then wire it into your statusline. Easiest from inside Claude Code:

```
/museline:setup
```

That seeds `~/.claude/museline/config.env` and adds a `statusLine` block to your
`~/.claude/settings.json` (backing it up first). Start a new session and enjoy.

### Manual wiring (if you prefer)

Add to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "museline-statusline",
    "refreshInterval": 2
  }
}
```

`museline-statusline` is on your PATH whenever the plugin is active. If your setup
does not resolve it, use the absolute path printed by `museline-setup`.

## Configure

Edit `~/.claude/museline/config.env` (created by `/museline:setup`; template in
[`config.env.example`](./config.env.example)):

```sh
MUSELINE_PERSONALITIES="philosophy trivia popculture witty"   # pick any subset
MUSELINE_ROTATE_SECS=4                                         # seconds per line
MUSELINE_LLM=0                                                 # 1 = generate with your LLM
MUSELINE_ICON="emoji"                                          # emoji | none | nerd
```

> **Emoji look like boxes?** Some terminals (notably Alacritty) do not render
> color emoji well. You have two fixes:
> - `MUSELINE_ICON="none"` shows clean plain-text lines.
> - `MUSELINE_ICON="nerd"` prefixes each line with a monochrome per-personality
>   [Nerd Font](https://www.nerdfonts.com) glyph (book for philosophy, lightbulb
>   for trivia, music note for pop-culture, magic wand for witty). They inherit
>   your theme color, exactly like Claude Code's own UI. Requires a Nerd Font.

Two intervals work together:

* `refreshInterval` in settings.json is how often Claude Code re-runs the script.
* `MUSELINE_ROTATE_SECS` is how often the line actually changes.

Keep `refreshInterval` less than or equal to `MUSELINE_ROTATE_SECS` so no rotation is
skipped.

### Bring your own LLM (optional)

Set these in `config.env` to blend in freshly generated lines:

```sh
MUSELINE_LLM=1
MUSELINE_PROVIDER="anthropic"            # or "openai"
MUSELINE_API_KEY_ENV="ANTHROPIC_API_KEY" # env var holding your key (or set MUSELINE_API_KEY)
# MUSELINE_MODEL="claude-haiku-4-5-20251001"
```

The statusline **never blocks on the network**. Generation runs detached in the
background, results are cached, and the curated packs are always the fallback. No
key, no network, or a bad key just means you get the curated packs.

## Add your own lines

Each pack in [`packs/`](./packs) is one display line per row (a leading emoji is
nice but optional). Edit them, or add a new `packs/<name>.txt` and include
`<name>` in `MUSELINE_PERSONALITIES`.

## How it works

* `bin/museline-statusline` is the fast display path. It reads the enabled packs plus
  the generated cache, and picks a line by a time-bucketed hash so it scatters
  across packs and stays stable within each tick. Pure bash, portable (no
  `mapfile`, no `jq`).
* `bin/museline-generate` is the background LLM refill (Anthropic/OpenAI). It dedupes
  and caps the cache, and fails silently.
* `bin/museline-setup` is the one-shot installer that seeds config and edits
  settings.json.

## License

MIT
