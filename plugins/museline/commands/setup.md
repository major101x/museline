---
description: Wire the museline rotating statusline into your Claude Code settings
allowed-tools: Bash(museline-setup), Bash(museline-setup:*), Read
---

Run the `museline-setup` helper to install the museline statusline for this user.

Steps:
1. Run `museline-setup` in the shell. (It seeds `~/.claude/museline/config.env` and wires
   `statusLine` into `~/.claude/settings.json`, backing up the file first.)
2. Show the user the command's output.
3. Tell them:
   - They can pick personalities and enable their own LLM by editing
     `~/.claude/museline/config.env` (packs: philosophy, trivia, popculture, witty).
   - The new statusline appears on the next session (or after a brief refresh).

Do not edit their settings.json by hand - the script handles it safely.
