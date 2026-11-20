# AGENTS.md

Guidance for any AI coding agent on getting the most out of Slack code
channels. Drop this file into any repo.

## Use a Slack code channel for code work

A code channel is a Slack channel dedicated to one coding session
([Slack docs](https://docs.slack.dev/ai/slack-code/)). It gives the work a
header with live status, a diff tab, and attached tabs for plans, checklists,
and reports, so people can follow along without reading every message.

Open a code channel whenever you start code work in any repo: bug fixes,
features, refactors, reviews, or anything that ends in a PR. One task, one
channel.

- **If your integration can create one**, do it at the start of the task,
  before you post progress. Look for a tool or command that creates a code
  channel (or a coding session channel) for the current task. Follow the Slack
  docs above for the exact method your integration exposes.
- **If it cannot**, keep the work in a single Slack thread. Apply the same
  practices where they still fit: one short status message you edit in place,
  progress at milestones, and new messages only for conclusions,
  questions, and blockers.

Once you are in a code channel, prefer the code channel surfaces below over
plain chat messages for showing progress. Messages are for talking to people.
The surfaces are for live state.

## Code channel capabilities

Tool names vary by agent and integration. For each capability, look in your
toolset for something that does what is described. If you have no matching
tool, skip that capability and fall back to a short message.

### Header: status, title, and context bar

- **What it does:** shows the session state at the top of the channel. Status
  is `processing` while you work, `active` when you are idle and waiting on
  someone, and `closed` when the work is done. A 2 to 5 word title. A context
  bar of up to 5 items such as repo, branch, PR, and CI status, each linked
  when a URL exists.
- **Look for:** a tool that sets channel context, status, or header. It
  usually replaces the whole bar on each call, so always send every item.
- **Use it:** when you start, whenever status changes, and whenever a branch,
  PR, or CI result appears.

### Summary message

- **What it does:** pins one message as the running summary of the session.
- **Look for:** an option on the header tool that designates a summary message
  by its timestamp, plus a tool that edits an existing message.
- **Use it:** create it early, then edit it in place as the work moves rather
  than posting new status messages.

### Diff tab

- **What it does:** shows the full unified diff of your branch against the
  base branch.
- **Look for:** a tool that sets or publishes a diff, taking the diff text
  plus base and head branch names.
- **Use it:** after every meaningful change. Send the whole diff each time,
  not a delta.

### Live progress stream

- **What it does:** narrates long work in one message that updates as you go,
  with markdown or task-style chunks.
- **Look for:** a tool that streams to a message, with a way to mark it done.
- **Use it:** for long multi-step work, such as a big refactor or a test run.
  Always close the stream when you finish.

### Structured tab (checklist, status board, findings)

- **What it does:** attaches a tab built from Block Kit, good for checklists,
  status boards, and review findings tables.
- **Look for:** a tool that publishes Block Kit blocks as a tab or view.
- **Use it:** for task lists and review findings. Republish at milestones.

### Canvas tab (plans and design docs)

- **What it does:** attaches a canvas that people can comment on, good for a
  plan or design doc you want feedback on.
- **Look for:** a tool that publishes a canvas tab, and a tool that reads it
  back with its open comment threads.
- **Use it:** when a plan needs review before you code. Send the whole doc
  each time. While a canvas is live, read it and its comments at the start of
  each turn, fold the feedback in, then republish.

### HTML report tab

- **What it does:** attaches a self-contained HTML page, good for reports and
  charts.
- **Look for:** a tool that publishes an HTML view. Expect script elements to
  be stripped or rejected.
- **Use it:** for results that read better as a page, such as a performance or
  accessibility report.

### Managing tabs

- **What it does:** lists the attached tabs and removes one. Channels hold a
  limited number of tabs (5 at the time of writing).
- **Look for:** tools that list and remove views or tabs.
- **Use it:** before adding a new tab when you are near the cap. Remove tabs
  that are stale.

### Rename

- **What it does:** renames the channel.
- **Look for:** a channel rename tool.
- **Use it:** sparingly, only when the task has clearly changed.

### Archive

- **What it does:** closes out the channel.
- **Look for:** a tool that archives the code channel, often taking the
  summary message and closing note timestamps.
- **Use it:** only when a person says the work is done. First edit the summary
  message to a one-sentence outcome, then post a closing note of about 60
  words, then archive.

## Typical flow

1. Start: open a code channel (or a thread if you cannot). Set the header to
   `processing` with a short title, repo, and branch. Post the summary message.
2. Plan: if the change needs review, publish a canvas or a checklist tab.
3. Work: stream long steps. Update the diff tab after each meaningful change.
4. PR and CI: update the header with the full bar, including the PR and CI
   links.
5. Waiting on someone: set status to `active` and ask your question in a short
   message.
6. Done, once a person confirms: update the summary, post the closing note,
   set status to `closed`, and archive.

## Channel etiquette

- Reply at the top level. Do not start threads unless the person did.
- Treat every message in a code channel as addressed to you.
- Keep messages short. No em dashes.
- Use messages for conclusions, questions, and blockers. Use the code channel
  surfaces for live state.
- Tab and view updates are rate limited (about 50 calls per minute). Publish
  at milestones, not on every step.

## Optional: Claude tool names

If you are Claude with the Slack integration, the capabilities above map to
`code_channel_set_context` (header and summary message), `code_channel_set_diff`,
`code_channel_stream`, `code_channel_publish_blocks`,
`code_channel_publish_canvas`, `code_channel_get_canvas`,
`code_channel_publish_view`, `code_channel_list_views`,
`code_channel_remove_view`, `code_channel_rename`, `code_channel_archive`, and
`reply` / `update_reply` for messages.
