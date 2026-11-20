# AGENTS.md

Guidance for AI coding agents working in this repo.

## Project

Welo Guard marketing homepage. Static HTML and CSS, no build step.

- Dev server: `npm run dev`
- Format: `npm run format`
- Check formatting: `npm run lint`

## Working from a Slack code channel

A code channel is a Slack channel created for one coding session
([docs](https://docs.slack.dev/ai/slack-code/)). When you run in one, favor the
code channel tools over plain chat messages for showing progress. The tools
hold live state (status, diff, plans, checklists) in the channel header and
tabs, so the conversation stays short and readable.

The tools are exposed with an `mcp__slackbot__` prefix. Bare names are used
below.

### Tools

- `code_channel_set_context`: set the channel header. Status is `processing`
  while working, `active` when idle and waiting, `closed` when done. Give a 2-5
  word title and a context bar of up to 5 items (repo, branch, PR, CI status),
  each with a URL when one exists. Every call replaces the whole bar. It can
  also designate one summary message (`summary_message_ts`); keep that message
  fresh with `update_reply`.
- `code_channel_set_diff`: publish the full unified diff against the base branch
  into the diff tab. Send the whole diff after every meaningful change, with
  `base_branch` and `head_branch`.
- `code_channel_stream`: narrate long work live in one streaming message, with
  markdown or task-card chunks. Finish with `done: true`.
- `code_channel_publish_blocks`: a Block Kit tab for a checklist, status board,
  or findings table. Republish at milestones.
- `code_channel_publish_canvas`: a comment-only canvas tab for a plan or design
  doc that people annotate. Send the whole doc each time.
- `code_channel_get_canvas`: read a canvas tab plus its open comment threads.
  Call it at the start of each turn while a canvas is live, fold the feedback
  in, then republish.
- `code_channel_publish_view`: a self-contained HTML tab for reports or charts.
  No `<script>` elements.
- `code_channel_list_views` / `code_channel_remove_view`: list the attached tabs
  or remove one. There is a 5-view cap.
- `code_channel_rename`: rename the channel. Use sparingly.
- `code_channel_archive`: archive the channel. Only when the user says the work
  is done (see the flow below).

### Typical flow

1. Start of work: `code_channel_set_context` with status `processing`, a short
   title, and the repo and branch.
2. While working: `code_channel_stream` for long tasks;
   `code_channel_publish_blocks` or `code_channel_publish_canvas` for plans and
   checklists.
3. After each meaningful change: `code_channel_set_diff` with the full diff.
4. PR opened or CI changes: `code_channel_set_context` again with the full bar
   (repo, branch, PR, CI).
5. Waiting on the user: set status to `active` and ask with `reply`.
6. Done, and the user says so: update the summary message to a one-sentence
   outcome, post a ~60-word closing note with `reply`, then call
   `code_channel_archive` with both ts values.

### Channel etiquette

- Reply at the top level. Don't thread unless the user did.
- Every message in the channel is addressed to you.
- Keep messages short. No em dashes.
- Use `reply` for conclusions, questions, and blockers. Use the code channel
  tools for live state.
- View tools are rate limited to about 50 calls per minute. Publish at
  milestones, not on every step.
