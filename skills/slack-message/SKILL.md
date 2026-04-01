---
name: slack-message
description: Craft and send beautifully formatted Slack messages using the Slack MCP. Use this skill whenever the user wants to send a Slack message of any kind — replying to a technical question in a thread, notifying a teammate about PRs or code changes, explaining how a system works, or any communication that should look polished and professional. If the user mentions Slack, a colleague they want to message, a thread they want to reply to, or a PR they want to share — use this skill. Don't wait for the user to ask explicitly for "Slack formatting" — if they're composing a message for Slack, this skill applies.
---

# Slack Message

You are helping an engineering lead craft Slack messages that are genuinely pleasant to read. The goal is not just to convey information — it's to make colleagues feel like they're getting a well-considered, thoughtfully structured message rather than a wall of text or a lazy one-liner.

The two main patterns are:
1. **Technical Q&A replies** — answering questions about how a service or system works
2. **PR notifications** — alerting a teammate to PRs that need their attention

The Slack MCP sends messages as markdown text — there is no Blocks API available. Structure comes from markdown formatting and emojis.

---

## Core Principles

**Lead with the answer.** Engineers are busy. Put the most important thing first. A blockquote TL;DR at the top of a technical reply is always welcome.

**Use visual hierarchy through markdown.** Bold labels and emojis act as section signposts. Lists break up dense content. Blockquotes (`>`) highlight the key takeaway. Don't make readers parse a wall of text.

**Emojis as signposts, not decoration.** A well-placed emoji tells the reader what kind of content follows before they read it. Use them consistently: 🔍 for explanations, 💡 for key insights/TL;DR, ⚠️ for caveats, 🔗 for links, ✅ for done items, 📋 for lists, 👀 for review requests, ⏱️ for timelines. One emoji per section/point is the sweet spot.

**Keep it tight.** Long messages get skimmed. Use lists over prose. If it's really long, consider whether a thread reply + link to docs is better.

---

## Markdown Formatting Reference

| Intent | Syntax |
|---|---|
| Bold | `**text**` |
| Italic | `_text_` |
| Inline code | `` `code` `` |
| Strikethrough | `~text~` |
| Code block | ` ```lang ... ``` ` |
| Blockquote | `> text` |
| Link | `[display text](https://url)` |
| Bullet list | `- item` |
| Numbered list | `1. item` |

---

## Template: Technical Q&A Reply

When someone asks how a service or system works, structure the reply to be immediately useful.

**Structure:**
1. Bold topic heading with emoji
2. Blockquote TL;DR (one-sentence bottom line up front)
3. Blank line, then numbered steps or bullet points for the detail
4. Code block if relevant
5. Caveat or follow-up note at the end if needed

**Example:**
```
🔍 **How the event pipeline works**

> 💡 **TL;DR:** Events are published to SQS, consumed by a Lambda worker, and written to DynamoDB — the API never writes to the DB directly.

**📋 Full flow:**
1. API receives the request and publishes an event to SQS
2. Lambda worker polls SQS and processes the message
3. Processed data is written to DynamoDB

```js
await sqs.sendMessage({ QueueUrl, MessageBody: JSON.stringify(event) });
```

⚠️ Failed events are retried 3× before being dead-lettered. 🔗 [Event Pipeline Docs](https://internal.docs/event-pipeline)
```

---

## Template: PR Notification

When alerting a colleague about PRs, be specific about what you need from them and why.

**Structure:**
1. Bold heading with emoji
2. Intro line — who it's for, what the work is. **If there's a linked ticket (Jira, Linear, GitHub issue), include it inline here** as a natural part of the sentence. Don't bury it.
3. One entry per PR: bold title + link, branch info on next line, brief description, specific callout if there's something to focus on
4. Timeline/urgency note at the end

**Tips:**
- If multiple PRs relate to each other, say so and whether order matters
- Be specific about what you want: "quick sanity check", "need approval to merge", "just FYI"
- Call out specific files or logic if there's something worth focusing on
- **Always ask for a ticket/issue link if the user hasn't provided one** — it saves the reviewer having to hunt for context

**Example:**
```
👀 **PR Review Request**

Hey @sarah! 👋 These two PRs are for the new checkout flow ([ENG-456](https://linear.app/team/issue/ENG-456)). They build on each other — review in order.

**1️⃣ [Add cart validation middleware](https://github.com/org/repo/pull/201)** ✅
`feature/cart-validation` → `main`
Validates cart contents before checkout proceeds. Worth looking at the out-of-stock edge cases in `cartMiddleware.ts` line 47.

**2️⃣ [Checkout summary page](https://github.com/org/repo/pull/202)** 🛒
`feature/checkout-summary` → `feature/cart-validation`
Depends on #1 — just needs a quick sanity check on the layout.

⏱️ Targeting merge by Thursday — no rush today 🙏
```

---

## Sending the Message

Both `slack_send_message` and `slack_send_message_draft` accept a single `message` parameter (markdown string).

- **To send immediately:** use `slack_send_message` with `channel_id` and `message`
- **To save a draft** for the user to review first: use `slack_send_message_draft`
- **To reply in a thread:** include `thread_ts` with the parent message timestamp

### Finding recipients
- Use `slack_search_users` to find a user's ID for DMs. If the search fails, ask the user to provide the ID directly.
- Use `slack_search_channels` to find a channel ID
- Use `slack_read_thread` to read context before composing a reply

### Gathering context before composing
- If replying to a question, read the thread first with `slack_read_thread`
- If the user mentions a PR, ask for the URL if not provided
- If the user hasn't specified a recipient, ask who to send to
- If sending a PR notification without a ticket link, ask for one
