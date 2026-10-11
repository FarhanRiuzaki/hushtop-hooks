# hushtop-hooks

The exact hooks [Hushtop](https://hushtop.app) adds to Claude Code and Codex, published so you can read them before you connect, and remove them by hand if you ever want to.

Hushtop is a Mac notch app. Its developer modules show every Claude Code and Codex session, the one that needs you first, and your limits, and let you answer approvals from the notch. All of that works through the hooks below. Nothing here phones home: the hooks write small JSON files into Hushtop's own folder, or talk to Hushtop over a local socket.

## What Hushtop writes

| File | Key | Purpose |
|---|---|---|
| `~/.claude/settings.json` | `hooks.Notification`, `hooks.Stop`, `hooks.UserPromptSubmit` | Session state: a turn started, Claude is waiting, Claude stopped. Each hook drops one JSON file in `~/Library/Application Support/Hushtop/inbox`. |
| `~/.claude/settings.json` | `hooks.PermissionRequest` | Answer from the notch. Runs Hushtop's helper, which shows the request in the notch and prints your decision back to Claude. |
| `~/.claude/settings.json` | `statusLine` | Limits in the notch. Wraps your existing status line, if you have one, and passes its text through. |
| `~/.codex/hooks.json` | `hooks.PermissionRequest` | The same approval hook for Codex, which reads hooks in Claude's shape. Codex asks you to trust a new hook once (`/hooks`). |

Every change is shown as a diff in Hushtop before it is written, a backup is kept, and Remove in Settings puts the file back. A file Hushtop created from scratch is emptied on Remove.

## How the approval hook behaves

```sh
if [ -x '/Applications/Hushtop.app/Contents/Helpers/mynotch-statusline' ]; then exec '…/mynotch-statusline' claude-ask; fi # mynotch-ask
```

- Hushtop running and the feature on: the request appears in the notch, you answer there, the answer goes back to Claude.
- Hushtop not running, the feature off, a timeout, or anything unexpected: the hook prints nothing and Claude asks in the terminal as usual.
- Never an automatic allow or deny. If you answer in the terminal first, Hushtop withdraws the question.
- Timeout 600 s for Claude Code. Codex shows nothing in its terminal while a hook waits, so its wait is shorter (120 s), then Codex asks as usual.

## What the inbox hooks record

Agent name, event, the project folder's name, the terminal app, the session id and the tty. No prompt text, no file contents, no transcript. The files stay in your own Library folder; nothing leaves the Mac.

## Files in this repo

- `examples/claude-settings.hooks.json`: the `hooks` block as Hushtop writes it (the `# mynotch-hook` and `# mynotch-ask` markers are how Hushtop finds its own entries again).
- `examples/claude-settings.statusline.json`: the `statusLine` entry.
- `examples/codex-hooks.json`: the Codex approval hook.

They are examples, not an installer. Hushtop merges its entries into the file you already have; it never replaces it.

## Remove by hand

Delete the entries whose `command` ends in `# mynotch-hook`, `# mynotch-ask`, `# mynotch-codex-ask` or `# mynotch-statusline`. If `hooks` or `statusLine` is then empty, you can drop the key.

## License

The snippets in this repo are MIT. Hushtop itself is a paid app: 7-day trial, then a one-time purchase. Apple silicon, macOS 15 or later. https://hushtop.app
