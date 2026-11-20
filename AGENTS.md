# AGENTS.md

Guidance for any AI coding agent on getting the most out of Slack code
channels. Drop this file into any repo.

Source: Slack's [Slack Code docs](https://docs.slack.dev/ai/slack-code/),
[code channel lifecycle](https://docs.slack.dev/ai/slack-code/lifecycle/), and
[code channel properties](https://docs.slack.dev/ai/slack-code/code-channel-properties/).

## Use a Slack code channel for code work

A code channel is a Slack channel dedicated to one agent task. It looks and
acts like a normal channel, plus a session status, a context bar (repo,
branch, PR, CI), and view tabs the agent manages (diff, HTML, Block Kit,
canvas). Code channels are semi-ephemeral: one channel per task, archived when
the work is done, and still searchable and reopenable afterwards.

Open a code channel for any code work in any repo: bug fixes, features,
refactors, reviews that need back-and-forth, or anything that ends in a PR.
Slack's rule of thumb: if the work takes more than one message (changing
code, running builds or tests, opening a PR, iterating with the requester),
open a channel automatically. Do not make the user ask for one.

- **Reply in place instead** when the answer fits in one message: a question,
  a lookup, a short explanation, or a quick review comment. If it is genuinely
  ambiguous, reply in place and offer to go deeper in a code channel.
- **Already in a code channel?** Do not create another one. Continue the
  session there. A channel is a code channel when `conversations.info` returns
  `properties.record_channel.record_type` of `agent_channel`. Users can also
  start code channels themselves from Slack's UI, so you may be added to one
  you did not create.

### How to create one

Look for a tool in your toolset that creates a code channel. Under the hood it
calls `agents.conversations.create` with:

- `origin_channel_id` and `origin_message_ts`: the message that asked for the
  work. Slack then matches the origin channel's privacy, lets its members
  join, names the channel from the message if you omit `name`, and posts a
  card in the origin conversation linking to the new channel.
- `name`: a short task name. Required only when there is no origin message.
- `session_id`: an idempotency key. Retrying with the same value returns the
  existing channel instead of creating a second one.

The app needs `features.code_channels.enabled: true` in its manifest and the
`code_channels:manage` scope. The `agents.sessions.*` methods need
`chat:write`.

### If you cannot create one

Use a session thread instead: the same status surface, scoped to one thread.
Call `agents.sessions.setStatus` with the thread's `thread_ts`. Keep the rest
of the practices below where they still fit: one status message edited in
place, progress at milestones, and new messages only for conclusions,
questions, and blockers.

## Code channel capabilities

Tool names vary by agent and integration. For each capability, look in your
toolset for something that does what is described. The Slack API method is
listed so you can match a tool to it. If you have no matching tool, skip the
capability and fall back to a short message.

Once you are in a code channel, prefer these surfaces over plain chat messages
for showing progress. Messages are for talking to people. The surfaces are for
live state.

### Session status

- **Slack API:** `agents.sessions.setStatus` (omit `thread_ts` in a code
  channel).
- **What it does:** shows where the session stands. Values:
  - `processing`: you are working. Slack shows a "Working..." indicator and a
    stop button.
  - `active`: you finished a turn with nothing outstanding.
  - `suspended`: you asked a question and need the answer to continue.
  - `closed`: the session is over. This does not archive the channel.
- **Use it:** set `processing` at the start of each turn and flip it when the
  turn ends. Channels are created as `active`. `processing` times out to
  `active` after one hour, so re-send it during long runs.

### Context bar

- **Slack API:** `agents.conversations.setProperties` with
  `code_channel.context_bar_items`.
- **What it does:** pins up to 5 items to the top of the channel, such as repo,
  branch, PR, and CI status. Each item has a `key`, `label`, optional `icon`,
  and optional `url`. An item can instead be an action button
  (`item_type: "action"`) that sends your app a `code_channel_action` event.
- **Use it:** at the start (repo, branch), and whenever a PR opens or CI
  changes. The array replaces the whole bar, so always resend every item you
  want to keep. Keep labels current, for example "PR #42 is merged" or "Tests
  passing".

### Session title

- **Slack API:** `agents.sessions.rename`.
- **What it does:** renames the channel and the session title together.
- **Use it:** only when the session's focus changes.

### Diff (Code tab)

- **Slack API:** `agents.conversations.setView` with `type: "diff"`.
- **What it does:** shows your branch's full unified diff (`git diff` output)
  in the channel's Code tab, with optional `base_branch` and `head_branch`
  labels. One diff per channel.
- **Use it:** after every meaningful change. Send the whole diff each time.
  Each call replaces the previous content.

### Live progress stream

- **Slack API:** `chat.startStream` to begin, `chat.stopStream` to finish.
- **What it does:** streams one message that fills in as you work. In a code
  channel, omit `thread_ts` so it posts at the top level. `chat.stopStream`
  can also set the session status (for example `suspended` when the stream
  ends on a question).
- **Use it:** for long multi-step work. Always stop the stream when you
  finish.

### Plans and design docs (canvas tab)

- **Slack API:** `agents.conversations.setView` with `type: "canvas"` and
  `access_level: "comment"` to attach it, `agents.conversations.getCanvas` to
  read it with its open comment threads, and
  `agents.conversations.setCanvasContent` to replace its content.
- **What it does:** a plan doc that people comment on while you stay the only
  author.
- **Use it:** when a plan needs review before you code. While the canvas is
  live, read the comments at the start of each turn, fold the feedback in,
  then write the revised doc back. `setCanvasContent` keeps comment anchors on
  sections you did not change.

### Structured tab (Block Kit)

- **Slack API:** `agents.conversations.setView` with `type: "block_kit"`.
- **What it does:** a tab built from Block Kit, good for checklists, status
  boards, and findings tables. Buttons and other inputs work as they do in
  messages.
- **Use it:** for task lists and review findings. Update it at milestones.

### Reports (HTML tab)

- **Slack API:** `agents.conversations.setView` with `type: "html"`.
- **What it does:** renders a complete, self-contained HTML document in a
  sandbox with a restricted Content-Security-Policy. Do not rely on external
  scripts or stylesheets. Extra `https` origins must be declared with the
  `csp` argument.
- **Use it:** for results that read better as a page, such as a test,
  coverage, or performance report.

### Managing tabs

- **Slack API:** `agents.conversations.listViews` and
  `agents.conversations.removeView`.
- **What it does:** lists the attached views, or removes one by `view_key` or
  `view_id`. A channel holds up to 5 views, at most 1 of them a diff. Each
  `setView` call with the same `view_key` updates that view in place.
- **Use it:** before adding a view when you are near the cap, and to remove
  stale ones. To link to a tab from a message, use a `rich_text` channel
  element with the view's `view_id` as `tab_id` (`"code"` for the Code tab).

### Slash commands

- **Slack API:** `agents.conversations.setCommands`.
- **What it does:** registers per-channel slash commands your agent offers,
  such as run tests or open a PR.
- **Use it:** when there are a few repeatable actions people will want to
  trigger.

### Archive

- **Slack API:** `agents.conversations.archive`, optionally with
  `summary_message_ts`.
- **What it does:** archives the channel for everyone. History, views, and the
  context bar are kept and stay searchable. `summary_message_ts` records your
  summary message on the channel and shares it back to the origin message (it
  requires the channel to have an origin link).
- **Use it:** when the work is finished (PR merged, task abandoned, or the user
  asks to wrap up). The recommended pattern: post a wrap-up summary, offer to
  archive, and archive with that summary's `ts` once the user confirms.

### Stop requests

- **Slack API:** the `agent_session_stopped` event.
- **What it does:** tells you the user pressed stop.
- **Use it:** stop all in-progress work for that channel or thread
  immediately, and do not post or call session or channel methods for it
  again until someone sends a new message.

## Typical flow

1. Start: someone asks for code work. Open a code channel from their message
   (or a session thread if you cannot). Set status to `processing`. Set the
   context bar with repo and branch.
2. Plan: if the change needs review, attach a canvas or a Block Kit checklist.
3. Work: stream long steps. Update the diff after each meaningful change.
4. PR and CI: resend the full context bar with the PR and CI items.
5. End of each turn: set status to `active`, or `suspended` if you are waiting
   on an answer.
6. Done: post a wrap-up summary, confirm with the user, then archive with the
   summary's `ts`.

## Channel etiquette

- Reply at the top level. Only thread when the user threaded first or you are
  answering a specific earlier message.
- Respond without being @-mentioned. In a code channel, messages are aimed at
  you. Use judgement: skip messages clearly meant for a human teammate.
- Keep messages short. No em dashes.
- Use messages for conclusions, questions, and blockers. Use the code channel
  surfaces for live state.
- Respect rate limits. `agents.sessions.*`, `getCanvas`, and
  `setCanvasContent` are Tier 3 (50+ per minute). `create`, `setProperties`,
  `setView`, `listViews`, `removeView`, and `archive` have special rate limits.
  Update views at milestones, not on every step, and honor `Retry-After` when
  you get `ratelimited`.

## Optional: Claude tool names

If you are Claude with the Slack integration, the capabilities above map to
`code_channel_set_context` (status, title, context bar, summary message),
`code_channel_set_diff`, `code_channel_stream`, `code_channel_publish_blocks`,
`code_channel_publish_canvas`, `code_channel_get_canvas`,
`code_channel_publish_view`, `code_channel_list_views`,
`code_channel_remove_view`, `code_channel_rename`, `code_channel_archive`, and
`reply` / `update_reply` for messages.
