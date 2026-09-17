---
description: <One line, imperative, what the user gets — this is what shows up in /help>
allowed-tools: mcp__wpcom__wpcom-mcp-site, mcp__wpcom__wpcom-mcp-content-authoring, mcp__plugin_<your-use-case>_wpcom__wpcom-mcp-site, mcp__plugin_<your-use-case>_wpcom__wpcom-mcp-content-authoring, AskUserQuestion, Skill
arguments:
  - name: <arg>
    description: <What it is, with an example, and what happens if it's omitted>
    required: false
---

# <Command Title>

<Two sentences: what this command does and when to reach for it.>

<Which skills to load, and before which step. Skill loading is probabilistic — naming the skill in
the command is what makes it reliable.>

## Step 1: Verify the connection

Call `action: list` on the relevant facade. If it fails with an authorization or connection error,
tell the user:

> I can't reach the WordPress.com MCP server. Enable MCP at WordPress.com → **Preferences → AI and
> MCP**, then run `/mcp` here and connect `wpcom`.

Then STOP.

## Step 2: <The gate, if this command has one>

<Any precondition that makes the rest of the command unsafe if unmet goes here, before any write.
Verify it by **reading state back**, not by trusting a write's return value. On failure, say exactly
what's wrong and which command fixes it. Then STOP.>

## Step 3: <Do the work>

<Numbered, explicit, in order. `describe` an operation before its first use in a session. Set status
explicitly on every create. Read writes back before reporting success.>

## Step 4: Report

<Show the user what actually happened, with read-back values rather than intentions. Then name the
next command they'd plausibly want.>

---

<Closing scope line: one sentence on what this command does and does not do.>

<If the use case has a safety or scope block, paste it here verbatim — the same text that's in every
SKILL.md. Do not paraphrase it to fit and do not factor it out; see CONTRIBUTING.md.>
