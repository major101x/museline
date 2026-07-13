# 🎭 museline

A rotating, personality-driven **statusline for Claude Code**. Instead of a plain
status row, museline cycles through short lines with real character while Claude
works:

```
🐋 Random fact: a bowhead whale can live over 200 years.
🤔 What is the meaning of it all? (still buffering an answer...)
🎤 Did you know? Kanye once spent a reported $10k+ on a single music video sofa.
🌀 Reticulating splines...
```

Four built-in **personality packs** (`philosophy`, `trivia`, `popculture`,
`witty`) get mixed and rotated live. Optionally, plug in **your own LLM API key**
and museline will generate fresh on-theme lines in the background.

> **Why a statusline and not the spinner word?** Claude Code's spinner verbs
> ("Enchanting...") are hard-coded and have no plugin hook. The statusline is the
> supported, update-proof surface: it re-runs on a timer (even mid-work), so
> museline rotates right alongside the spinner. Your line is the star; the
> built-in verb just keeps it company.

## Install

```
/plugin marketplace add major101x/museline
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
[`plugins/museline/config.env.example`](./plugins/museline/config.env.example)):

```sh
MUSELINE_PERSONALITIES="philosophy trivia popculture witty"   # pick any subset
MUSELINE_ROTATE_SECS=4                                        # seconds per line
MUSELINE_LLM=0                                                # 1 = generate with your LLM
MUSELINE_ICON="emoji"                                         # emoji | none | nerd
MUSELINE_COLOR=""                                            # "#rrggbb" tint, e.g. "#d97757"
```

Set `MUSELINE_COLOR` to a `#rrggbb` hex to tint the line (truecolor terminals);
`"#d97757"` is Claude's signature orange, matching the loading-spinner accent.
Color emoji keep their own colors; plain text and Nerd Font glyphs take the tint.

Claude Code has no vertical-padding setting, so if the line looks cramped, use
`MUSELINE_PAD_TOP` and `MUSELINE_PAD_BOTTOM` (a count of blank rows) to add
breathing room above and below.

> **Emoji showing as boxes?** That is a terminal/font limitation (common in
> Alacritty), not a bug. See
> [Icons, emoji, and your terminal](#icons-emoji-and-your-terminal) for the fix.

Two intervals work together:

* `refreshInterval` in settings.json is how often Claude Code re-runs the script.
* `MUSELINE_ROTATE_SECS` is how often the line actually changes.

Keep `refreshInterval` less than or equal to `MUSELINE_ROTATE_SECS` so no rotation
is skipped.

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

Each pack in [`plugins/museline/packs/`](./plugins/museline/packs) is one display
line per row (a leading emoji is nice but optional). Edit them, or add a new
`packs/<name>.txt` and include `<name>` in `MUSELINE_PERSONALITIES`.

## Icons, emoji, and your terminal

museline leads each line with an icon. How well it renders depends on your
terminal and font, so there are three modes, set with `MUSELINE_ICON`:

| Mode | What you get | Best for |
|------|--------------|----------|
| `emoji` (default) | Color emoji as written in the packs | Terminals with good color-emoji support (kitty, WezTerm, foot, iTerm2, most GUI terminals) |
| `nerd` | A monochrome per-personality Nerd Font glyph that inherits your theme color | Anyone using a [Nerd Font](https://www.nerdfonts.com), including Alacritty |
| `none` | Plain text, no leading icon | Any terminal, zero font requirements |

### Why emoji can look broken

Color emoji (like the whale in a "Random fact" line) are **bitmap** glyphs that
carry their own color and live in a dedicated emoji font such as Noto Color Emoji.
Some terminals, most notably **Alacritty**, do not scale or render those bitmap
color fonts well, so the emoji shows up as an empty box ("tofu") or a blank space.
By contrast, the icons Claude Code uses in its own UI are monochrome **vector**
glyphs from the loaded text font, which is why they always render and take the
theme color.

This is a terminal/font limitation, not a museline bug. Three ways to fix it:

1. **Use a terminal with solid color-emoji rendering.** kitty, WezTerm, foot, and
   iTerm2 all handle emoji cleanly. Keep `MUSELINE_ICON="emoji"`.
2. **Switch to Nerd Font glyphs.** Set `MUSELINE_ICON="nerd"`. These are the same
   kind of monochrome vector glyphs Claude Code uses for its icons: they live in
   your text font, render anywhere the Nerd Font is loaded, and pick up your theme
   color. Default mapping: book (philosophy), lightbulb (trivia), music note
   (pop-culture), magic wand (witty), and star (LLM-generated lines).
3. **Drop icons entirely.** Set `MUSELINE_ICON="none"` for plain text that works
   in any terminal.

`MUSELINE_ICON` supersedes the older `MUSELINE_EMOJI` flag (`1` = emoji, `0` =
none), which is still honored when `MUSELINE_ICON` is unset.

## How it works

* `bin/museline-statusline` is the fast display path. It reads the enabled packs
  plus the generated cache, and picks a line by a time-bucketed hash so it
  scatters across packs and stays stable within each tick. Pure bash, portable
  (no `mapfile`, no `jq`).
* `bin/museline-generate` is the background LLM refill (Anthropic/OpenAI). It
  dedupes and caps the cache, and fails silently.
* `bin/museline-setup` is the one-shot installer that seeds config and edits
  settings.json.

## Repository layout

```
.claude-plugin/marketplace.json   # makes this repo an installable marketplace
plugins/museline/                 # the plugin itself
├── .claude-plugin/plugin.json
├── bin/                          # museline-statusline, museline-generate, museline-setup
├── commands/setup.md             # the /museline:setup command
├── packs/                        # philosophy, trivia, popculture, witty
├── config.env.example
└── README.md
```

## License

[MIT](./LICENSE)
