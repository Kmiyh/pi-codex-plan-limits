# @kmiyh/pi-codex-plan-limits

`@kmiyh/pi-codex-plan-limits` is a Pi extension that shows your **OpenAI Codex subscription limits** directly in Pi's footer.

It is designed specifically for people who use Pi with the **`openai-codex` subscription provider** and want to see, at a glance:

- how much of the **5-hour Codex window** is left
- how much of the **weekly Codex window** is left
- when each limit **resets**

Instead of showing less useful subscription cost/context footer segments like:

```text
$0.000 (sub) 0.0%/272k (auto)
```

this extension replaces that segment with Codex limit information such as:

```text
5h 45% ▰▰▰▰▱▱▱▱▱▱ reset 19:07   Weekly 1% ▱▱▱▱▱▱▱▱▱▱ reset Apr 20 11:00
```

---

## What this project adds to Pi

After installation, the extension modifies the **footer** in interactive Pi sessions.

### When the extension is active

If Pi is currently using:

- provider: **`openai-codex`**
- authentication mode: **OAuth/subscription auth**

then the footer keeps the normal Pi information, but replaces the subscription-specific cost/context segment with:

- `5h` remaining percentage
- `5h` reset time
- `Weekly` remaining percentage
- `Weekly` reset time
- compact progress indicators for both windows

### When the extension is not active

If Pi is **not** using an `openai-codex` model, or it is using that provider **without subscription/OAuth auth**, the extension disables itself and Pi's footer returns to the **default behavior exactly as before**.

That means:

- no custom Codex limits are shown
- the original Pi footer is restored
- nothing extra remains on screen

This is intentional.

---

## Why this extension exists

OpenAI exposes Codex usage limits in its own interfaces, but Pi does not show those limits in a way that is convenient during everyday use.

When using Pi with the Codex subscription provider, the built-in footer can still show token/context style information that is less relevant than the actual Codex subscription limits.

This extension makes the footer more useful for that workflow.

---

## How it works

## Source of truth

The extension uses **Pi's own authentication state**.

It does **not** depend on a separate Codex CLI login.
It does **not** read `~/.codex/sessions`.
It does **not** scrape any external UI.

Instead, it uses:

- the currently active Pi model
- Pi's auth storage for the `openai-codex` provider
- the same Codex usage backend endpoint used for the subscription session

Current usage endpoint:

```text
https://chatgpt.com/backend-api/wham/usage
```

So the shown limits are tied to the **same account and subscription Pi is currently using**.

## Fallback behavior

If a live refresh fails, the extension keeps the **last successful snapshot fetched through Pi** and marks it as cached/stale internally.

It does not import external Codex CLI state.

---

## Footer behavior

The extension preserves the normal Pi footer structure as much as possible.

### Preserved

The following footer information stays in place:

- current project path / cwd
- git branch
- session name
- token counters such as `↑`, `↓`, `R`, `W`
- active model name
- provider label when relevant
- thinking level
- extension statuses

### Replaced

Only the subscription-specific segment that is not very helpful for Codex subscription usage gets replaced.

In other words, the extension is intentionally **surgical**.

---

## Display format

Current limit format:

```text
[label] [percent] [micro-bar] reset [time]
```

Examples:

```text
5h 45% ▰▰▰▰▱▱▱▱▱▱ reset 19:07
Weekly 1% ▱▱▱▱▱▱▱▱▱▱ reset Apr 20 11:00
```

### Time formatting rules

For the **5-hour limit**:

- if reset is today, the extension shows only the time:
  - `19:07`
- if needed, it can also show short weekday-based formatting for nearer dates

For the **weekly limit**:

- the extension uses a more explicit month/day + time format:
  - `Apr 20 11:00`

This makes the weekly reset easier to scan without ambiguity.

---

## Update frequency

The extension refreshes limit data automatically.

### Automatic refresh triggers

1. **On session start**
2. **When the active model changes**
3. **After a turn ends**
4. **Periodic polling every 60 seconds**

### Debounce / throttling

To avoid excessive refreshes, turn-based refreshes are debounced:

- minimum refresh gap: **15 seconds** for event-driven refreshes
- polling interval: **60 seconds**

So in practice, the data is usually refreshed:

- immediately when you start or switch into Codex subscription usage
- then at most about once a minute during ongoing work
- and also after meaningful session events when enough time has passed

---

## Visibility rules

The extension shows Codex limits **only** when all of the following are true:

- Pi is running in interactive UI mode
- the active model provider is `openai-codex`
- Pi is using **OAuth/subscription auth** for that model

If any of those is false, the extension restores the default Pi footer.

---

## Commands

| Command | Description |
| --- | --- |
| `/codex-limits` | Force an immediate refresh of the displayed Codex limits |

---

## Installation

```bash
pi install npm:@kmiyh/pi-codex-plan-limits
```

For local development / testing from this repository:

```bash
pi -e ./src/index.ts
```

---

## Local development

Install dependencies:

```bash
npm install
```

Run typecheck:

```bash
npm run typecheck
```

---

## Technical notes

### Why the extension shows percentages instead of “messages left”

OpenAI's Codex limits are not well represented by a simple fixed message counter.

Actual usage depends on factors like:

- model choice
- task size
- context size
- local vs cloud execution patterns

Because of that, a fake “N requests left” number would be misleading.

Percent remaining is the safest and clearest representation.

### Why the extension replaces only part of the footer

The goal is not to redesign Pi's footer.
The goal is to improve one weak spot for Codex subscription usage.

So the extension only swaps out the part that is least useful in this specific workflow.

---

## Limitations

- only intended for Pi sessions using `openai-codex`
- requires Pi to have valid OAuth/subscription auth for that provider
- if the Pi auth session expires, refreshes can fail until `/login` refreshes it
- currently focused on the main **5h** and **Weekly** Codex windows
- does not currently expose every possible future metered limit bucket

---

## Security

This extension reads Pi's local OpenAI Codex OAuth credentials only to request usage data from OpenAI's official backend.

It does **not**:

- send tokens anywhere except OpenAI's own backend endpoint
- read external Codex CLI session history
- log your access token

---

## Future ideas

Possible future improvements:

- even better compact footer formatting
- optional alternate display modes
- optional status command with richer breakdown
- support for additional Codex metered buckets if OpenAI exposes them
- user-tunable polling interval

---

## Publish workflow

This repository uses the same publish workflow as `repo-pi-skills-menu`:

- tag-based release
- npm publish
- GitHub Packages publish

See [PUBLISHING.md](./PUBLISHING.md).

---

## License

MIT
